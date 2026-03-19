# 🔄 Многопоточность

=== "Swift 6"
    ```swift
    // Swift 6 строго проверяет thread-safety на этапе компиляции!

    // DispatchQueue (GCD) — низкоуровневый API
    let bgQueue = DispatchQueue(label: "com.app.bg", qos: .userInitiated)
    let concurrent = DispatchQueue(label: "com.app.concurrent",
                                   attributes: .concurrent)

    bgQueue.async {
        let data = heavyComputation()
        DispatchQueue.main.async { updateUI(data) }
    }

    // sync — блокирует текущий поток до завершения
    let result = bgQueue.sync { compute() }

    // DispatchGroup — дождаться группы задач
    let group = DispatchGroup()
    for url in urls {
        group.enter()
        download(url) { group.leave() }
    }
    group.notify(queue: .main) { print("All downloads done") }

    // Semaphore
    let semaphore = DispatchSemaphore(value: 3)
    for _ in 0..<10 {
        bgQueue.async {
            semaphore.wait()
            defer { semaphore.signal() }
            performLimitedWork()
        }
    }

    // Mutex (Swift 5.9+ — import Synchronization)
    import Synchronization
    let lock = Mutex(0)
    lock.withLock { state in state += 1 }

    // Noncopyable types (Swift 6) — гарантия единственного владельца
    struct FileHandle: ~Copyable {
        private let fd: Int32
        init(path: String) { fd = open(path, O_RDONLY) }
        consuming func readAll() -> Data { ... }
        deinit { close(fd) }
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Thread (низкоуровневый — редко нужен напрямую)
    val thread = Thread {
        println("Thread: ${Thread.currentThread().name}")
    }.also { it.start() }
    thread.join()

    // ExecutorService (из Java)
    import java.util.concurrent.*

    val executor = Executors.newFixedThreadPool(4)
    val future: Future<Int> = executor.submit<Int> { compute() }
    val result = future.get(5, TimeUnit.SECONDS)
    executor.shutdown()

    // Atomic операции — lock-free, быстрее mutex
    import java.util.concurrent.atomic.*

    val counter = AtomicInteger(0)
    counter.incrementAndGet()                       // thread-safe ++
    counter.compareAndSet(1, 2)                    // CAS операция

    val ref = AtomicReference<User?>(null)
    ref.compareAndSet(null, User(1, "Alice"))

    // ConcurrentHashMap — thread-safe map
    val map = ConcurrentHashMap<String, Int>()
    map.compute("key") { _, v -> (v ?: 0) + 1 }

    // Coroutines — РЕКОМЕНДУЕТСЯ вместо Thread (см. раздел Async)
    val mutex = Mutex()
    suspend fun safeIncrement(counter: AtomicInteger) {
        mutex.withLock { counter.incrementAndGet() }
    }

    // Structured concurrency
    suspend fun robustFetch(ids: List<Int>): List<User> = coroutineScope {
        ids.map { id -> async(Dispatchers.IO) { fetchUser(id) } }
           .awaitAll()
    }
    ```

=== "Python 3.13"
    ```python
    # threading — OS-потоки (ограничены GIL для CPU задач, хороши для I/O)
    import threading
    from threading import Thread, Lock, RLock, Semaphore, Event

    results: list[str] = []
    lock = Lock()

    def worker(name: str) -> None:
        data = do_work(name)
        with lock:
            results.append(data)

    threads = [Thread(target=worker, args=(f"t{i}",), daemon=True) for i in range(5)]
    for t in threads: t.start()
    for t in threads: t.join()

    # RLock — reentrant lock
    rlock = RLock()
    def recursive_work(n: int) -> None:
        with rlock:
            if n > 0: recursive_work(n - 1)

    # Event — сигнал между потоками
    event = Event()
    def producer() -> None:
        time.sleep(1)
        event.set()

    def consumer() -> None:
        event.wait()
        print("Got signal!")

    # multiprocessing — обходит GIL (настоящая параллельность для CPU)
    from multiprocessing import Pool
    import multiprocessing as mp

    def cpu_heavy(n: int) -> int:
        return sum(i**2 for i in range(n))

    with Pool(processes=mp.cpu_count()) as pool:
        results = pool.map(cpu_heavy, [10**6] * 4)

    # concurrent.futures — высокоуровневый API (РЕКОМЕНДУЕТСЯ)
    from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed

    # I/O задачи — ThreadPoolExecutor
    with ThreadPoolExecutor(max_workers=20) as ex:
        futures = {ex.submit(fetch_url, url): url for url in urls}
        for future in as_completed(futures):
            url = futures[future]
            try:
                result = future.result(timeout=5)
            except Exception as e:
                print(f"{url}: {e}")

    # CPU задачи — ProcessPoolExecutor
    with ProcessPoolExecutor(max_workers=mp.cpu_count()) as ex:
        results = list(ex.map(cpu_heavy, range(8), chunksize=2))

    # Python 3.13 — Free-Threaded mode (PEP 703, experimental)
    # Установка: python3.13t  или  python --disable-gil
    # Отключает GIL → настоящая параллельность в Thread без subprocess
    ```
