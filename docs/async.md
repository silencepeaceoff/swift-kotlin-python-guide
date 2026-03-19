# ⚡ Асинхронное программирование

=== "Swift 6"
    ```swift
    // async/await + Structured Concurrency (Swift 5.5+, строгость Swift 6)

    struct User: Codable { let id: Int; let name: String }

    // async throws — асинхронная функция, которая может выбросить ошибку
    func fetchUser(id: Int) async throws -> User {
        let url = URL(string: "https://jsonplaceholder.typicode.com/users/\(id)")!
        let (data, response) = try await URLSession.shared.data(from: url)
        guard let http = response as? HTTPURLResponse, http.statusCode == 200 else {
            throw URLError(.badServerResponse)
        }
        return try JSONDecoder().decode(User.self, from: data)
    }

    // Точка входа для async кода
    Task {
        do {
            let user = try await fetchUser(id: 1)
            print(user.name)
        } catch {
            print("Error: \(error)")
        }
    }

    // async let — запускает несколько задач ПАРАЛЛЕЛЬНО
    async let user  = fetchUser(id: 1)
    async let posts = fetchUser(id: 2)           // оба запроса идут одновременно!
    let (u, p) = try await (user, posts)

    // TaskGroup — динамическое количество параллельных задач
    func fetchAllUsers(ids: [Int]) async throws -> [User] {
        try await withThrowingTaskGroup(of: User.self) { group in
            for id in ids {
                group.addTask { try await fetchUser(id: id) }
            }
            var users: [User] = []
            for try await user in group { users.append(user) }
            return users
        }
    }

    // Task приоритеты и отмена
    let task = Task(priority: .userInitiated) {
        try await fetchUser(id: 1)
    }
    task.cancel()
    // Внутри задачи: try Task.checkCancellation()

    // Actor — потокобезопасный изолированный state (Swift 6)
    actor BankAccount {
        private var balance: Double = 0

        func deposit(_ amount: Double) { balance += amount }

        func withdraw(_ amount: Double) throws {
            guard balance >= amount else { throw BankError.insufficient }
            balance -= amount
        }

        var currentBalance: Double { balance }
    }

    let account = BankAccount()
    await account.deposit(100)                    // await обязателен для actor

    // @MainActor — гарантирует выполнение на главном потоке
    @MainActor
    class ViewModel: ObservableObject {
        @Published var items: [User] = []
        @Published var isLoading = false

        func load() async {
            isLoading = true
            defer { isLoading = false }
            items = try! await fetchAllUsers(ids: Array(1...5))
        }
    }

    // AsyncSequence — асинхронная версия Sequence
    func streamNumbers() -> AsyncStream<Int> {
        AsyncStream { continuation in
            Task {
                for i in 1...10 {
                    try? await Task.sleep(for: .milliseconds(100))
                    continuation.yield(i)
                }
                continuation.finish()
            }
        }
    }

    for await number in streamNumbers() {
        print(number)
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Coroutines (kotlinx.coroutines)
    import kotlinx.coroutines.*
    import kotlinx.coroutines.flow.*

    // suspend функция — как async в Swift
    suspend fun fetchUser(id: Int): User {
        delay(100)                                    // неблокирующая задержка
        return User(id, "Alice")
    }

    // Coroutine Builders
    fun main() = runBlocking {                        // блокирует поток — только для main/tests
        val user = fetchUser(1)
        println(user)
    }

    // launch — fire-and-forget, возвращает Job
    val job = CoroutineScope(Dispatchers.IO).launch {
        val user = fetchUser(1)
        withContext(Dispatchers.Main) { updateUI(user) }
    }
    job.cancel()
    job.join()

    // async/await — для получения результата (Deferred<T>)
    val deferred: Deferred<User> = CoroutineScope(Dispatchers.IO).async {
        fetchUser(1)
    }
    val user = deferred.await()

    // Параллельные запросы через async + await
    coroutineScope {
        val u = async { fetchUser(1) }
        val p = async { fetchPosts(1) }
        println("${u.await()}, ${p.await()}")
    }

    // Dispatchers
    // Dispatchers.Main    — UI поток (Android / JavaFX)
    // Dispatchers.IO      — I/O: файлы, сеть, БД (до 64 потоков)
    // Dispatchers.Default — CPU: вычисления (N CPU потоков)

    // Flow — холодный реактивный поток (аналог AsyncSequence)
    fun numberFlow(): Flow<Int> = flow {
        for (i in 1..10) {
            delay(100)
            emit(i)
        }
    }

    numberFlow()
        .filter { it % 2 == 0 }
        .map { it * it }
        .onEach { println("Processing $it") }
        .collect { println("Result: $it") }

    // StateFlow — горячий, всегда имеет значение (аналог @Published)
    class ViewModel {
        private val _state = MutableStateFlow<UiState>(UiState.Loading)
        val state: StateFlow<UiState> = _state.asStateFlow()

        fun load() = viewModelScope.launch {
            _state.value = UiState.Loading
            _state.value = try {
                UiState.Success(fetchUser(1))
            } catch (e: Exception) {
                UiState.Error(e.message ?: "Unknown")
            }
        }
    }

    // SharedFlow — горячий broadcast (много подписчиков)
    val events = MutableSharedFlow<Event>(replay = 0, extraBufferCapacity = 64)
    events.emit(Event.Click)

    // Mutex и Semaphore для coroutines
    val mutex = Mutex()
    suspend fun safeUpdate() = mutex.withLock { updateSharedState() }

    val semaphore = Semaphore(3)
    suspend fun limitedWork() = semaphore.withPermit { doWork() }
    ```

=== "Python 3.13"
    ```python
    import asyncio
    import aiohttp
    from dataclasses import dataclass

    @dataclass
    class User:
        id: int
        name: str

    # coroutine — определяется через async def
    async def fetch_user(session: aiohttp.ClientSession, user_id: int) -> User:
        async with session.get(
            f"https://jsonplaceholder.typicode.com/users/{user_id}"
        ) as resp:
            resp.raise_for_status()
            data = await resp.json()
            return User(id=data["id"], name=data["name"])

    # Точка входа
    async def main() -> None:
        async with aiohttp.ClientSession() as session:
            user = await fetch_user(session, 1)
            print(user)

    asyncio.run(main())

    # Параллельные задачи через asyncio.gather
    async def fetch_all() -> list[User]:
        async with aiohttp.ClientSession() as session:
            tasks = [fetch_user(session, i) for i in range(1, 6)]
            users = await asyncio.gather(*tasks, return_exceptions=False)
            return list(users)

    # TaskGroup (Python 3.11+) — лучше gather по обработке ошибок
    async def fetch_with_group() -> tuple[User, User]:
        async with aiohttp.ClientSession() as session:
            async with asyncio.TaskGroup() as tg:
                t1 = tg.create_task(fetch_user(session, 1))
                t2 = tg.create_task(fetch_user(session, 2))
            # Если одна задача упала — вторая отменяется автоматически!
            return t1.result(), t2.result()

    # asyncio.timeout (Python 3.11+)
    async def with_timeout() -> User | None:
        async with aiohttp.ClientSession() as session:
            try:
                async with asyncio.timeout(5.0):
                    return await fetch_user(session, 1)
            except TimeoutError:
                return None

    # async generator — ленивый асинхронный итератор
    async def paginate(session: aiohttp.ClientSession, url: str):
        page = 1
        while True:
            async with session.get(f"{url}?page={page}") as resp:
                data = await resp.json()
                if not data: break
                for item in data: yield item
                page += 1

    # Queue — producer-consumer паттерн
    async def producer(queue: asyncio.Queue[int]) -> None:
        for i in range(10):
            await queue.put(i)
            await asyncio.sleep(0.1)
        await queue.put(None)

    async def consumer(queue: asyncio.Queue[int]) -> None:
        while (item := await queue.get()) is not None:
            print(f"Processing: {item}")
            queue.task_done()

    # Синхронизация
    lock = asyncio.Lock()
    async def safe_update() -> None:
        async with lock:
            await update_shared_resource()

    sem = asyncio.Semaphore(3)
    async def limited_work() -> None:
        async with sem:
            await do_work()
    ```
