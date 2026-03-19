# 🧠 Функциональное программирование

=== "Swift 6"
    ```swift
    let numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]

    let evens    = numbers.filter { $0 % 2 == 0 }
    let doubled  = numbers.map { $0 * 2 }
    let sum      = numbers.reduce(0, +)
    let strings  = numbers.compactMap { n -> String? in n > 5 ? "\(n)" : nil }
    let flat     = [[1,2],[3,4],[5]].flatMap { $0 }

    // Цепочки (lazy — не создаёт промежуточные массивы)
    let result = (1...1_000_000)
        .lazy
        .filter { $0 % 2 == 0 }
        .map { $0 * $0 }
        .prefix(5)
        .reduce(0, +)

    // Функции высшего порядка
    func compose<A, B, C>(_ f: @escaping (A) -> B,
                           _ g: @escaping (B) -> C) -> (A) -> C {
        { g(f($0)) }
    }
    let doubleAndStr = compose({ $0 * 2 }, { "\($0)" })
    doubleAndStr(5)                                // "10"

    // Currying (каррирование)
    func add(_ x: Int) -> (Int) -> Int { { y in x + y } }
    let add5 = add(5)
    [1,2,3].map(add5)                              // [6,7,8]

    // KeyPath как функция (Swift 5.2+)
    struct Person { let name: String; let age: Int }
    let people = [Person(name: "Alice", age: 30), Person(name: "Bob", age: 25)]
    people.map(\.name)                             // ["Alice", "Bob"]
    people.sorted(using: KeyPathComparator(\.age))
    ```

=== "Kotlin 2.x"
    ```kotlin
    val numbers = listOf(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)

    val evens   = numbers.filter { it % 2 == 0 }
    val doubled = numbers.map { it * 2 }
    val sum     = numbers.reduce { acc, n -> acc + n }
    val sum2    = numbers.sum()
    val strings = numbers.mapNotNull { if (it > 5) "$it" else null }
    val flat    = listOf(listOf(1,2), listOf(3,4)).flatten()

    // Sequence — ленивые цепочки
    val result = (1..1_000_000)
        .asSequence()
        .filter { it % 2 == 0 }
        .map { it * it }
        .take(5)
        .sum()

    // Группировка и агрегация
    val byParity = numbers.groupBy { if (it % 2 == 0) "even" else "odd" }
    val (evens2, odds) = numbers.partition { it % 2 == 0 }
    numbers.windowed(3)                            // [[1,2,3],[2,3,4],...]
    numbers.chunked(3)                             // [[1,2,3],[4,5,6],...]
    numbers.zipWithNext { a, b -> b - a }         // разности соседних

    // Currying
    fun add(x: Int): (Int) -> Int = { y -> x + y }
    val add5 = add(5)
    listOf(1,2,3).map(add5)                       // [6,7,8]

    // Member references (аналог KeyPath)
    data class Person(val name: String, val age: Int)
    val people = listOf(Person("Alice", 30), Person("Bob", 25))
    people.map(Person::name)                       // ["Alice", "Bob"]
    people.sortedBy(Person::age)
    ```

=== "Python 3.13"
    ```python
    from functools import reduce, partial, lru_cache
    from itertools import takewhile, dropwhile, groupby, chain, islice, starmap, accumulate
    from operator import add, mul, attrgetter, itemgetter

    numbers = list(range(1, 11))

    # Comprehensions — pythonic functional style
    evens   = [x for x in numbers if x % 2 == 0]
    doubled = [x * 2 for x in numbers]
    flat    = [item for sub in [[1,2],[3,4]] for item in sub]

    # map / filter / reduce
    sum_all = reduce(add, numbers, 0)

    # itertools — ленивые операции
    first5_evens = list(islice(filter(lambda x: x % 2 == 0, range(10**9)), 5))

    # takewhile / dropwhile
    ascending = list(takewhile(lambda x: x < 5, numbers))  # [1,2,3,4]

    # chain — объединить итерируемые
    combined = list(chain([1,2], [3,4], [5,6]))

    # starmap — map с распаковкой tuple
    pairs = [(1, 10), (2, 20), (3, 30)]
    list(starmap(add, pairs))                     # [11, 22, 33]

    # accumulate — накопленные суммы
    list(accumulate(numbers))                     # [1,3,6,10,15,21,28,36,45,55]

    # partial — частичное применение
    def power(base, exp): return base ** exp
    square = partial(power, exp=2)
    list(map(square, [1,2,3,4,5]))               # [1,4,9,16,25]

    # lru_cache — мемоизация
    @lru_cache(maxsize=None)
    def fibonacci(n: int) -> int:
        if n < 2: return n
        return fibonacci(n-1) + fibonacci(n-2)

    fibonacci(100)                                # мгновенно
    fibonacci.cache_info()                        # CacheInfo(hits=98, misses=101, ...)

    # Функциональная композиция
    def compose(*funcs):
        def composed(x):
            for f in reversed(funcs):
                x = f(x)
            return x
        return composed

    double = lambda x: x * 2
    inc    = lambda x: x + 1
    double_then_inc = compose(inc, double)
    double_then_inc(5)                            # 11

    # attrgetter / itemgetter — быстрые key functions
    people = [{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}]
    sorted(people, key=itemgetter("age"))
    ```
