# 🔧 Операторы и специальный синтаксис

=== "Swift 6"
    ```swift
    let a = 10, b = 3
    a + b; a - b; a * b
    a / b                                          // 3 (целочисленное)
    a % b                                          // 1
    Double(a) / Double(b)                          // 3.333

    // Сравнение
    a == b; a != b; a < b; a > b; a <= b; a >= b

    // Логические
    true && false; true || false; !true

    // Nil coalescing
    let val: Int? = nil
    val ?? 0                                       // 0

    // Optional chaining
    user?.address?.city

    // Range операторы
    1...5                                          // ClosedRange: 1,2,3,4,5
    1..<5                                          // Range: 1,2,3,4
    ...5                                           // PartialRangeThrough
    5...                                           // PartialRangeFrom

    // Тернарный
    let abs = a > 0 ? a : -a

    // Пользовательские операторы
    infix operator ** : MultiplicationPrecedence
    func ** (base: Double, exp: Double) -> Double { pow(base, exp) }
    2.0 ** 10.0                                    // 1024.0

    // Identity (=== / !==) — сравнение ссылок (только для class)
    let obj1 = MyClass(); let obj2 = obj1
    obj1 === obj2                                  // true (один объект)
    obj1 !== MyClass()                             // true (разные)
    ```

=== "Kotlin 2.x"
    ```kotlin
    val a = 10; val b = 3

    // Специфичные для Kotlin
    a.div(b)                                       // 3 — метод-аналог /
    a.rem(b)                                       // 1 — метод-аналог %
    a.and(b); a.or(b); a.xor(b); a.shl(1); a.shr(1)

    // Elvis
    val y: Int? = null
    val z = y ?: 0
    val w = y ?: throw IllegalStateException()

    // Диапазоны
    1..5                                           // IntRange: 1,2,3,4,5
    1 until 5                                      // 1,2,3,4
    5 downTo 1                                     // 5,4,3,2,1
    1..10 step 2                                   // 1,3,5,7,9

    // in / !in
    5 in 1..10                                     // true
    "hello" !in listOf("a", "b")                  // true

    // Spread оператор *
    fun sum(vararg n: Int) = n.sum()
    val arr = intArrayOf(1, 2, 3)
    sum(*arr)

    // Destructuring
    val (first, second, third) = listOf(1, 2, 3)
    val (key, value) = mapEntry

    // Scope functions
    val sb = StringBuilder().apply {
        append("Hello")
        append(", World!")
    }
    val len = "hello".let { it.length }
    val same = listOf(1,2,3).also { println(it) }
    with(config) { println(host); println(port) }

    // Оператор перегрузка
    data class Vec(val x: Int, val y: Int) {
        operator fun plus(o: Vec)  = Vec(x+o.x, y+o.y)
        operator fun times(n: Int) = Vec(x*n, y*n)
        operator fun unaryMinus()  = Vec(-x, -y)
        operator fun get(i: Int)   = if (i == 0) x else y
    }
    Vec(1,2) + Vec(3,4)                            // Vec(4,6)
    ```

=== "Python 3.13"
    ```python
    a, b = 10, 3
    a + b; a - b; a * b
    a // b                                         # 3 (floor division)
    a % b                                          # 1
    a / b                                          # 3.333 (всегда float)
    a ** b                                         # 1000 (степень)

    # Логические (слова, не символы!)
    True and False; True or False; not True

    # Цепочки сравнения (уникально для Python!)
    0 < a < 100                                    # True — как в математике!

    # Walrus operator := (Python 3.8+) — присваивание в выражении
    if (n := len(data)) > 10:
        print(f"Too long: {n}")

    while chunk := file.read(1024):
        process(chunk)

    # is / is not — идентичность (не равенство!)
    a is None
    a is not None
    # НЕ используй is для чисел > 256 и строк!

    # in / not in
    5 in [1, 2, 3, 4, 5]
    "key" in {"key": 1}
    "ell" in "Hello"

    # Перегрузка операторов через dunder-методы
    class Vector:
        def __init__(self, x: float, y: float):
            self.x, self.y = x, y

        def __add__(self, other: "Vector") -> "Vector":
            return Vector(self.x + other.x, self.y + other.y)

        def __mul__(self, n: float) -> "Vector":
            return Vector(self.x * n, self.y * n)

        def __rmul__(self, n: float) -> "Vector":   # 3 * v
            return self.__mul__(n)

        def __neg__(self) -> "Vector":
            return Vector(-self.x, -self.y)

        def __abs__(self) -> float:
            return (self.x**2 + self.y**2) ** 0.5

        def __getitem__(self, i: int) -> float:
            return [self.x, self.y][i]

        def __iter__(self):
            yield self.x; yield self.y

        def __repr__(self) -> str:
            return f"Vector({self.x}, {self.y})"

    v1 = Vector(1, 2)
    v2 = Vector(3, 4)
    v1 + v2                                        # Vector(4, 6)
    v1 * 3; 3 * v1                                # Vector(3, 6)
    abs(v1)                                        # 2.236...
    x, y = v1                                      # распаковка через __iter__
    ```
