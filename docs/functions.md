# 🧩 Функции

=== "Swift 6"
    ```swift
    // Базовое объявление — аргументные метки ОБЯЗАТЕЛЬНЫ при вызове
    func greet(name: String, greeting: String = "Hello") -> String {
        "\(greeting), \(name)!"             // implicit return в однострочных функциях
    }
    greet(name: "Alice")                    // Hello, Alice!
    greet(name: "Bob", greeting: "Hi")

    // Внешняя метка vs внутреннее имя
    func move(from start: Int, to end: Int) -> Int { end - start }
    move(from: 3, to: 10)                   // читается как предложение

    // Подавить метку через _
    func square(_ n: Int) -> Int { n * n }
    square(5)                               // без метки

    // Несколько возвращаемых значений (tuple с метками)
    func minMax(_ arr: [Int]) -> (min: Int, max: Int) {
        (arr.min()!, arr.max()!)
    }
    let result = minMax([3, 1, 4, 1, 5])
    print(result.min, result.max)

    // Variadic
    func sum(_ nums: Int...) -> Int { nums.reduce(0, +) }
    sum(1, 2, 3, 4, 5)

    // inout — передача по ссылке
    func doubleIt(_ x: inout Int) { x *= 2 }
    var val = 5; doubleIt(&val)             // val = 10

    // Функции как first-class citizens
    let add: (Int, Int) -> Int = { $0 + $1 }
    let operations: [(Int, Int) -> Int] = [add, { $0 - $1 }, { $0 * $1 }]

    // Trailing closure syntax
    [1,2,3,4,5].sorted { $0 > $1 }
    [1,2,3].map { $0 * 2 }

    // @discardableResult
    @discardableResult
    func save() -> Bool { return true }

    // Typed throws (Swift 6)
    enum DBError: Error { case connectionFailed, queryFailed(String) }
    func fetchUsers() throws(DBError) -> [User] {
        throw .connectionFailed
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Базовое объявление
    fun greet(name: String, greeting: String = "Hello"): String =
        "$greeting, $name!"

    greet("Alice")                          // позиционный аргумент
    greet(name = "Bob", greeting = "Hi")   // именованный

    // Unit = void (отсутствие значения, писать необязательно)
    fun logMessage(message: String) { println(message) }

    // Vararg
    fun sum(vararg nums: Int): Int = nums.sum()
    sum(1, 2, 3, 4, 5)
    val arr = intArrayOf(1, 2, 3)
    sum(*arr)                               // spread operator

    // Деструктуризация через Pair / data class
    fun minMax(arr: List<Int>): Pair<Int, Int> =
        arr.min() to arr.max()
    val (min, max) = minMax(listOf(3, 1, 4, 1, 5))

    // Extension functions — добавить метод к любому типу
    fun String.isPalindrome(): Boolean = this == this.reversed()
    "racecar".isPalindrome()                // true

    fun Int.clamp(min: Int, max: Int) = this.coerceIn(min, max)
    42.clamp(0, 10)                         // 10

    // Higher-order functions
    fun <T, R> transform(value: T, block: (T) -> R): R = block(value)
    transform(5) { it * 2 }                // 10

    // Inline functions — избегает аллокации lambda-объекта
    inline fun <T> measure(block: () -> T): T {
        val start = System.nanoTime()
        val result = block()
        println("${(System.nanoTime() - start) / 1_000_000}ms")
        return result
    }
    val result = measure { heavyComputation() }

    // Operator overloading
    data class Vector(val x: Int, val y: Int) {
        operator fun plus(other: Vector) = Vector(x + other.x, y + other.y)
        operator fun times(n: Int) = Vector(x * n, y * n)
    }
    Vector(1, 2) + Vector(3, 4)            // Vector(4, 6)
    ```

=== "Python 3.13"
    ```python
    # Базовое объявление с аннотациями типов
    def greet(name: str, greeting: str = "Hello") -> str:
        return f"{greeting}, {name}!"

    greet("Alice")
    greet("Bob", greeting="Hi")

    # *args и **kwargs
    def total(*nums: int) -> int:
        return sum(nums)

    def describe(**kwargs: str) -> str:
        return ", ".join(f"{k}={v}" for k, v in kwargs.items())

    # keyword-only аргументы (после *)
    def process(data: list[int], *, verbose: bool = False, limit: int = 100) -> dict:
        return {"sum": sum(data[:limit]), "count": len(data)}

    process([1,2,3], verbose=True)         # verbose — keyword-only

    # positional-only аргументы (до /)
    def fast_add(x: int, y: int, /) -> int:   # x, y — positional-only
        return x + y

    # Lambda (только одно выражение)
    add = lambda x, y: x + y
    sorted_words = sorted(words, key=lambda w: w.lower())

    # Декораторы — фундамент Python
    from functools import wraps
    import time

    def timer(func):
        @wraps(func)                        # сохраняет метаданные оригинальной функции
        def wrapper(*args, **kwargs):
            start = time.perf_counter()
            result = func(*args, **kwargs)
            print(f"{func.__name__}: {time.perf_counter()-start:.4f}s")
            return result
        return wrapper

    def retry(times: int = 3):             # декоратор с параметрами
        def decorator(func):
            @wraps(func)
            def wrapper(*args, **kwargs):
                for attempt in range(times):
                    try:
                        return func(*args, **kwargs)
                    except Exception as e:
                        if attempt == times - 1: raise
                        print(f"Attempt {attempt+1} failed: {e}")
            return wrapper
        return decorator

    @timer
    @retry(times=3)
    def unreliable_api_call(): ...

    # Генераторы — ленивые последовательности
    def fibonacci():
        a, b = 0, 1
        while True:
            yield a
            a, b = b, a + b

    def take(n: int, gen):
        return [next(gen) for _ in range(n)]

    take(10, fibonacci())                  # [0,1,1,2,3,5,8,13,21,34]
    ```
