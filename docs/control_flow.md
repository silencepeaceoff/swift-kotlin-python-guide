# 🔀 Управление потоком

## Условия

=== "Swift 6"
    ```swift
    let x = 42

    // Стандартный if-else
    if x > 0 { print("positive") }
    else if x == 0 { print("zero") }
    else { print("negative") }

    // if как выражение (Swift 5.9+)
    let label = if x > 0 { "positive" } else { "non-positive" }

    // guard — ранний выход (избегай pyramid of doom)
    func process(value: Int?) {
        guard let v = value, v > 0 else { return }
        // v доступен здесь и до конца функции
        print("Processing \(v)")
    }

    // guard let shorthand (Swift 5.7+)
    func handle(value: Int?) {
        guard let value else { return }
        // value доступен как non-optional
        print(value)
    }

    // switch — exhaustive (компилятор требует все кейсы)
    switch x {
    case 0:
        print("zero")
    case 1...10:
        print("small")
    case let n where n < 0:
        print("negative: \(n)")
    case _ where x.isMultiple(of: 2):
        print("large even")
    default:
        print("other")
    }

    // switch как выражение (Swift 5.9+)
    let desc = switch x {
        case 0:      "zero"
        case 1...10: "small"
        default:     "large"
    }

    // Optional binding
    let opt: Int? = 42
    if let val = opt { print(val) }              // обычный unwrap
    if let opt { print(opt) }                    // shorthand (Swift 5.7+)
    let val = opt ?? 0                           // nil coalescing
    let forced = opt!                            // force unwrap — краш если nil!

    // Optional chaining
    struct User { var address: Address? }
    struct Address { var city: String }
    let city = user.address?.city ?? "Unknown"
    ```

=== "Kotlin 2.x"
    ```kotlin
    val x = 42

    // if — является выражением
    val label = if (x > 0) "positive" else "non-positive"
    val abs = if (x >= 0) x else -x

    // when — мощная замена switch (тоже выражение!)
    when (x) {
        0          -> println("zero")
        in 1..10   -> println("small")
        !in 0..100 -> println("out of range")
        else       -> println("large: $x")
    }

    val description = when {              // без аргумента — цепочка условий
        x < 0    -> "negative"
        x == 0   -> "zero"
        x < 100  -> "small"
        else     -> "large"
    }

    // Smart cast: is проверяет тип И кастует автоматически
    fun describe(obj: Any) = when (obj) {
        is Int    -> "Int: $obj"
        is String -> "String length: ${obj.length}"  // obj уже String здесь!
        is List<*>-> "List size: ${obj.size}"
        null      -> "null"
        else      -> "Unknown: ${obj::class.simpleName}"
    }

    // Elvis operator (аналог ??)
    val name: String? = null
    val display = name ?: "Anonymous"

    // Safe call + Elvis
    val length = name?.length ?: 0

    // let (для выполнения блока если не null)
    name?.let { println("Name is: $it") }

    // also, apply, run, with, let — scope functions
    val result = listOf(1,2,3)
        .also { println("Original: $it") }      // возвращает сам объект
        .map { it * 2 }
        .filter { it > 2 }
        .let { it.sum() }                       // трансформирует в другой тип
    ```

=== "Python 3.13"
    ```python
    x = 42

    # if-elif-else
    if x > 0:
        print("positive")
    elif x == 0:
        print("zero")
    else:
        print("negative")

    # Тернарный оператор
    label = "positive" if x > 0 else "non-positive"

    # match-case (Python 3.10+ — structural pattern matching)
    match x:
        case 0:
            print("zero")
        case int() as n if n < 0:              # guard
            print(f"negative: {n}")
        case int() as n if 1 <= n <= 10:
            print("small")
        case _:                                # default
            print(f"other: {x}")

    # match с деструктуризацией структур
    point = (1, 0)
    match point:
        case (0, 0): print("origin")
        case (x, 0): print(f"x-axis at {x}")
        case (0, y): print(f"y-axis at {y}")
        case (x, y): print(f"point ({x}, {y})")

    # match с классами (dataclass)
    from dataclasses import dataclass

    @dataclass
    class Command:
        action: str
        target: str

    cmd = Command("open", "file.txt")
    match cmd:
        case Command(action="open", target=t): print(f"Opening: {t}")
        case Command(action="delete"):         print("Deleting")
        case _:                                print("Unknown command")

    # match со списком
    match [1, 2, 3]:
        case []:          print("empty")
        case [x]:         print(f"one: {x}")
        case [x, y]:      print(f"two: {x}, {y}")
        case [x, *rest]:  print(f"first: {x}, rest: {rest}")
    ```

---

## Циклы

=== "Swift 6"
    ```swift
    for i in 0..<5  { print(i) }                 // 0,1,2,3,4
    for i in 0...5  { print(i) }                 // 0,1,2,3,4,5
    for i in stride(from: 0, to: 10, by: 2) { print(i) }   // 0,2,4,6,8
    for i in (1...5).reversed() { print(i) }    // 5,4,3,2,1

    let arr = [10, 20, 30]
    for (index, value) in arr.enumerated() {
        print("\(index): \(value)")
    }
    for value in arr where value > 15 { print(value) }   // with filter!

    var n = 0
    while n < 5 { n += 1 }
    repeat { n -= 1 } while n > 0             // do-while аналог

    // labeled break (выход из вложенного цикла)
    outer: for i in 0..<3 {
        for j in 0..<3 {
            if j == 1 { break outer }
        }
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    for (i in 0 until 5)          { print(i) }  // 0,1,2,3,4
    for (i in 0..5)               { print(i) }  // 0,1,2,3,4,5
    for (i in 10 downTo 0 step 2) { print(i) }  // 10,8,6,4,2,0

    val arr = listOf(10, 20, 30)
    for ((index, value) in arr.withIndex()) { println("$index: $value") }
    arr.forEach { println(it) }
    arr.forEachIndexed { i, v -> println("$i: $v") }

    var n = 0
    while (n < 5) { n++ }
    do { n-- } while (n > 0)

    repeat(3) { println("hello $it") }          // встроенная утилита

    // labeled break/continue
    outer@ for (i in 0..3) {
        for (j in 0..3) {
            if (j == 1) break@outer
            if (j == 0) continue@outer
        }
    }
    ```

=== "Python 3.13"
    ```python
    for i in range(5):        print(i)           # 0,1,2,3,4
    for i in range(0, 10, 2): print(i)           # 0,2,4,6,8
    for i in range(10, 0, -1): print(i)          # 10,9,...,1
    for i in reversed(range(5)): print(i)        # 4,3,2,1,0

    arr = [10, 20, 30]
    for i, v in enumerate(arr):       print(f"{i}: {v}")
    for i, v in enumerate(arr, start=1): print(f"{i}: {v}")

    # Итерация по нескольким спискам
    for a, b in zip([1,2,3], ["a","b","c"]): print(a, b)
    # zip_longest — если длины разные
    from itertools import zip_longest
    for a, b in zip_longest([1,2], ["a","b","c"], fillvalue=0): print(a, b)

    n = 0
    while n < 5: n += 1

    # for-else (уникальная особенность Python)
    for i in range(10):
        if i == 5: break
    else:
        print("Цикл завершился без break")   # НЕ выполнится (был break)

    # Comprehension вместо цикла (быстрее и pythonic)
    squares  = [x**2 for x in range(10)]
    evens    = [x for x in range(20) if x % 2 == 0]
    matrix   = [[i*j for j in range(5)] for i in range(5)]   # вложенный

    # Generator expression — ленивая версия (не грузит в память)
    total = sum(x**2 for x in range(1000000))   # не создаёт список!
    ```
