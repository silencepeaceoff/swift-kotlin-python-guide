# 📦 Переменные, Числа и Строки

## Объявление переменных

=== "Swift 6"
    ```swift
    // let = константа (immutable), var = переменная (mutable)
    let name: String = "Alice"
    let age = 30               // вывод типа (type inference)
    var score: Int = 0
    score = 100

    // Optional — переменная, которая МОЖЕТ быть nil (нет null-pointer краша)
    var nickname: String?       // == Optional<String>, значение nil по умолчанию
    var city: String! = nil     // Implicitly Unwrapped Optional — избегай в новом коде!

    // Базовые типы
    // Int, Int8/16/32/64, UInt, Float, Double, Bool, Character, String
    let flag: Bool = true
    let pi: Double = 3.14159

    // Числа с разделителями для читаемости
    let population = 8_000_000_000

    // Type alias (Swift 6)
    typealias Seconds = Double
    let timeout: Seconds = 30.0
    ```

=== "Kotlin 2.x"
    ```kotlin
    // val = read-only (константная ссылка), var = изменяемая
    val name: String = "Alice"
    val age = 30               // вывод типа
    var score: Int = 0
    score = 100

    // Null safety — прямо в системе типов
    var nickname: String? = null  // nullable — может быть null
    // val x: String = null       // ОШИБКА КОМПИЛЯЦИИ — String не может быть null

    // Базовые типы: Int, Long, Short, Byte, Float, Double, Boolean, Char, String
    // В JVM они компилируются в примитивы там где возможно (без boxing overhead)
    val flag: Boolean = true
    val pi: Double = 3.14159
    val bigNum = 8_000_000_000L   // L = Long

    // Type alias
    typealias Seconds = Double
    val timeout: Seconds = 30.0
    ```

=== "Python 3.13"
    ```python
    # Динамическая типизация — нет ключевых слов val/var/let
    # Аннотации типов не обязательны в runtime, но нужны для mypy/pyright
    name: str = "Alice"
    age = 30

    # Константы: соглашение UPPER_CASE (не настоящие константы — Python не запрещает изменение)
    MAX_SIZE: int = 1000

    # None — аналог null/nil
    nickname: str | None = None  # Python 3.10+ (Union type)

    # Базовые типы: int, float, complex, bool, str, bytes, NoneType
    # bool — подкласс int: True == 1, False == 0
    flag: bool = True
    pi: float = 3.14159

    # Python int имеет произвольную точность (BigInteger из коробки)
    huge = 10 ** 100             # googol — нет переполнения!

    # Числа с разделителями
    population = 8_000_000_000

    # Type alias (Python 3.12+ — новый синтаксис)
    type Seconds = float
    timeout: Seconds = 30.0
    ```

### Хранение в памяти

| Концепция | Swift | Kotlin | Python |
|-----------|-------|--------|--------|
| Value types | `struct`, `enum`, базовые типы (стек / inline в куче) | `Int`, `Boolean` и др. компилируются как JVM-примитивы | — (всё является объектом) |
| Reference types | `class` (ARC на куче) | `class`, все коллекции (JVM heap) | Все объекты (`int`, `str`, `list` — на куче) |
| Управление памятью | ARC (Automatic Reference Counting) — детерминированно | JVM GC (G1 / ZGC / Shenandoah) | CPython: подсчёт ссылок + циклический GC (`gc` модуль) |
| Null-safety | Compile-time (`Optional`) | Compile-time (`?`) | Runtime (`None`) |
| Копирование | Value types копируются при присваивании (COW-оптимизация) | Объекты передаются по ссылке | Все объекты — по ссылке; числа/строки — immutable |

!!! note "CPython Integer Caching"
    CPython кэширует объекты `int` в диапазоне −5 … 256 и интернирует короткие строки-идентификаторы. Это означает, что `a = 256; b = 256; a is b` вернёт `True` (один объект), а `a = 257; b = 257; a is b` — `False` (два разных объекта). Для сравнения значений всегда используй `==`, а не `is`.

---

## 🔢 Числа и операции

=== "Swift 6"
    ```swift
    let a: Int = 10
    let b: Int = 3

    // Арифметика
    let sum  = a + b          // 13
    let diff = a - b          // 7
    let prod = a * b          // 30
    let div  = a / b          // 3  — целочисленное деление!
    let rem  = a % b          // 1
    let fdiv = Double(a) / Double(b)  // 3.333...

    // Битовые операции
    let and = a & b            // 2
    let or  = a | b            // 11
    let xor = a ^ b            // 9
    let shl = a << 1           // 20
    let shr = a >> 1           // 5

    // Математика
    import Foundation
    let power   = pow(Double(a), Double(b))  // 1000.0
    let root    = sqrt(Double(a))            // 3.162...
    let absVal  = abs(-5)                    // 5
    let rounded = Double(a).rounded()        // 10.0

    // Overflow-безопасные операторы (не краш, а wrap-around)
    let overflow = Int.max &+ 1              // Int.min

    // Проверка диапазона
    let clamped = max(0, min(a, 5))         // 5
    ```

=== "Kotlin 2.x"
    ```kotlin
    val a: Int = 10
    val b: Int = 3

    val sum  = a + b                         // 13
    val div  = a / b                         // 3 — целочисленное!
    val fdiv = a.toDouble() / b             // 3.333...
    val rem  = a % b                         // 1

    import kotlin.math.*
    val power   = a.toDouble().pow(b.toDouble())  // 1000.0
    val root    = sqrt(a.toDouble())              // 3.162...
    val absVal  = abs(-5)                         // 5
    val rounded = round(3.7)                      // 4.0

    // Удобные extension-функции
    val clamped = a.coerceIn(0, 5)          // 5
    val maxVal  = maxOf(a, b, 7)            // 10

    // Infinity и NaN (как в Java)
    val inf = Double.POSITIVE_INFINITY
    val nan = Double.NaN
    nan.isNaN()                             // true
    ```

=== "Python 3.13"
    ```python
    a: int = 10
    b: int = 3

    s   = a + b            # 13
    d   = a // b           # 3  — floor division (всегда floor!)
    fd  = a / b            # 3.333... — всегда float!
    r   = a % b            # 1
    pw  = a ** b           # 1000 — возведение в степень (оператор, не функция)

    import math
    root   = math.sqrt(a)          # 3.162...
    ceil   = math.ceil(3.2)        # 4
    floor  = math.floor(3.8)       # 3
    log10  = math.log10(100)       # 2.0
    absVal = abs(-5)               # 5
    inf    = math.inf
    nan    = math.nan
    math.isnan(nan)                # True

    # divmod — частное и остаток за одну операцию
    q, r = divmod(a, b)            # (3, 1)

    # round (банковское округление в Python!)
    round(2.5)                     # 2 — к ближайшему чётному (banker's rounding)
    round(3.5)                     # 4

    # Decimal для точной арифметики (финансы и т.д.)
    from decimal import Decimal
    Decimal("0.1") + Decimal("0.2") == Decimal("0.3")  # True (в отличие от float!)
    ```

---

## 🔤 Строки

=== "Swift 6"
    ```swift
    let s = "Hello, World!"

    // Интерполяция
    let name = "Alice"
    let greeting = "Hello, \(name)! Age: \(30 * 2)"

    // Многострочные строки (отступ берётся от закрывающих кавычек)
    let multiline = """
        Line 1
        Line 2
        """

    // Основные операции
    s.count                                        // 13 (Unicode-корректно)
    s.uppercased()                                 // "HELLO, WORLD!"
    s.lowercased()                                 // "hello, world!"
    s.contains("World")                            // true
    s.hasPrefix("Hello")                           // true
    s.hasSuffix("!")                               // true
    s.replacingOccurrences(of: "World", with: "Swift")
    s.trimmingCharacters(in: .whitespaces)

    // Сплит / джойн
    let words = s.components(separatedBy: ", ")
    let joined = words.joined(separator: " - ")

    // Индексирование (через Index — сложнее, но Unicode-корректно!)
    let start = s.startIndex
    let end = s.index(start, offsetBy: 5)
    let sub = s[start..<end]                       // "Hello"

    // Удобный substring
    let prefix5 = String(s.prefix(5))             // "Hello"
    let suffix6 = String(s.suffix(6))             // "World!"

    // Raw String (без экранирования)
    let raw = #"Path: C:\Users\name\n (это буква n, не перевод строки)"#

    // Regex (Swift 5.7+ — нативный DSL)
    import RegexBuilder
    let regex = /\d+/
    if let match = s.firstMatch(of: /\d+/) { print(match.output) }

    // Regex Builder — типобезопасный
    let dateRegex = Regex {
        Capture { OneOrMore(.digit) }
        "-"
        Capture { OneOrMore(.digit) }
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    val s = "Hello, World!"

    // Интерполяция
    val name = "Alice"
    val greeting = "Hello, $name! Age: ${30 * 2}"

    // Многострочные строки
    val multiline = """
        Line 1
        Line 2
        """.trimIndent()     // убирает общий отступ

    // Основные операции
    s.length                                       // 13
    s.uppercase()                                  // "HELLO, WORLD!"
    s.lowercase()                                  // "hello, world!"
    s.contains("World")                            // true
    s.startsWith("Hello")                          // true
    s.endsWith("!")                                // true
    s.replace("World", "Kotlin")
    s.trim()                                       // убрать пробелы

    // Сплит / джойн
    val words = s.split(", ")                      // List<String>
    val joined = words.joinToString(" - ")

    // Подстрока
    val sub  = s.substring(0, 5)                   // "Hello"
    val sub2 = s.take(5)                           // "Hello" (удобнее)
    val sub3 = s.drop(7)                           // "World!"

    // Проверки
    s.isBlank()                                    // false
    s.isEmpty()                                    // false
    s.matches(Regex("\\w+"))

    // Regex
    val regex = Regex("""\d+""")
    regex.find(s)?.value
    regex.findAll(s).toList()

    // Форматирование строк (Java-style)
    "Value: %d, Float: %.2f".format(42, 3.14)
    ```

=== "Python 3.13"
    ```python
    s = "Hello, World!"

    # f-строки (Python 3.12+ — поддерживают вложенные кавычки и выражения)
    name = "Alice"
    greeting = f"Hello, {name}! Age: {30 * 2}"
    debug    = f"Repr: {name!r}"                   # repr()
    padded   = f"{42:>10}"                         # выравнивание: '        42'
    floatfmt = f"{3.14159:.2f}"                    # '3.14'
    # Python 3.12+: кавычки внутри {} без ограничений
    nested = f"Keys: {', '.join(['a', 'b', 'c'])}"

    # Многострочные строки
    multiline = """
    Line 1
    Line 2
    """
    import textwrap
    clean = textwrap.dedent("""
        Line 1
        Line 2
    """).strip()

    # Основные операции
    len(s)                                         # 13
    s.upper(); s.lower()
    "World" in s                                   # True
    s.startswith("Hello"); s.endswith("!")
    s.replace("World", "Python")
    s.strip(); s.lstrip(); s.rstrip()
    s.center(20, "-")                              # "---Hello, World!----"

    # Сплит / джойн
    words = s.split(", ")
    joined = " - ".join(words)

    # Срезы (slicing) — мощнейший инструмент Python
    sub   = s[0:5]                                 # "Hello"
    rev   = s[::-1]                                # "!dlroW ,olleH"
    every = s[::2]                                 # "Hlo ol!"

    # str.removeprefix / removesuffix (Python 3.9+)
    "Hello World".removeprefix("Hello ")           # "World"

    # Regex
    import re
    match = re.search(r"(\w+), (\w+)", s)
    if match:
        print(match.group(1), match.group(2))      # "Hello" "World"
        print(match.groups())                      # ('Hello', 'World')

    # Компиляция regex (для многократного использования)
    pattern = re.compile(r"\d+", re.IGNORECASE)
    pattern.findall("abc 123 def 456")             # ['123', '456']
    ```
