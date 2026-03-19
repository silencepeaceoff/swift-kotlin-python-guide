# 🏛️ ООП: Классы, структуры, протоколы

## Классы

=== "Swift 6"
    ```swift
    // class = reference type, управляется ARC
    class Animal {
        var name: String
        private(set) var energy: Int = 100        // снаружи read-only, внутри rw

        var isAlive: Bool { energy > 0 }          // вычисляемое свойство

        // Property observer
        var hunger: Int = 0 {
            willSet { print("Will change to \(newValue)") }
            didSet  { print("Changed from \(oldValue)") }
        }

        // Lazy свойство (инициализируется при первом обращении)
        lazy var heavyData: [Int] = { Array(0..<10000) }()

        required init(name: String) {             // required = обязателен в subclass
            self.name = name
        }

        func eat(amount: Int) { energy += amount }

        open func speak() -> String {             // open = можно переопределить вне модуля
            "\(name) makes a sound"
        }

        deinit { print("\(name) deallocated") }   // вызывается при освобождении ARC
    }

    // Наследование
    class Dog: Animal {
        var breed: String

        init(name: String, breed: String) {
            self.breed = breed
            super.init(name: name)
        }

        override func speak() -> String { "\(name) says Woof!" }

        final func fetch() { print("Fetching!") }  // нельзя переопределить
    }

    // Проверка типов
    let animal: Animal = Dog(name: "Rex", breed: "Lab")
    animal is Dog                               // true
    (animal as? Dog)?.breed                    // Optional("Lab")
    (animal as! Dog).breed                     // "Lab" — краш если не Dog
    ```

=== "Kotlin 2.x"
    ```kotlin
    // классы по умолчанию final (нельзя наследовать без open!)
    open class Animal(val name: String) {       // primary constructor

        private var energy: Int = 100

        val isAlive: Boolean get() = energy > 0

        var hunger: Int = 0
            set(value) {
                println("Hunger changing to $value")
                field = value                   // field — backing field
            }

        val heavyData: List<Int> by lazy { (0..9999).toList() }  // lazy через делегат

        init {
            println("$name created")
        }

        fun eat(amount: Int) { energy += amount }

        open fun speak(): String = "$name makes a sound"
    }

    class Dog(name: String, val breed: String) : Animal(name) {

        override fun speak(): String = "$name says Woof!"

        // Secondary constructor
        constructor(name: String) : this(name, "Mixed")
    }

    // data class — автогенерация equals/hashCode/toString/copy/componentN
    data class Point(val x: Double, val y: Double)
    val p1 = Point(1.0, 2.0)
    val p2 = p1.copy(y = 5.0)                  // Point(1.0, 5.0)
    p1 == Point(1.0, 2.0)                      // true (по значению!)
    val (x, y) = p1                            // деструктуризация

    // object — singleton (thread-safe из коробки)
    object AppConfig {
        val apiUrl = "https://api.example.com"
        fun isDebug() = BuildConfig.DEBUG
    }

    // companion object — "статические" члены класса
    class MathUtils {
        companion object {
            const val PI = 3.14159
            fun square(x: Int) = x * x
        }
    }
    MathUtils.square(5)                        // 25
    ```

=== "Python 3.13"
    ```python
    class Animal:
        # Атрибут класса (разделяется между экземплярами!)
        kingdom: str = "Animalia"
        _count: int = 0

        def __init__(self, name: str) -> None:
            self.name = name
            self._energy: int = 100
            Animal._count += 1

        @property
        def is_alive(self) -> bool:
            return self._energy > 0

        @is_alive.setter
        def is_alive(self, value: bool) -> None:
            self._energy = 100 if value else 0

        def eat(self, amount: int) -> None:
            self._energy += amount

        def speak(self) -> str:
            return f"{self.name} makes a sound"

        @staticmethod
        def classify(weight: float) -> str:
            return "large" if weight > 100 else "small"

        @classmethod
        def count(cls) -> int:
            return cls._count

        # Dunder-методы (magic methods)
        def __repr__(self) -> str:
            return f"Animal(name={self.name!r})"

        def __str__(self) -> str:
            return self.name

        def __eq__(self, other: object) -> bool:
            if not isinstance(other, Animal): return NotImplemented
            return self.name == other.name

        def __hash__(self) -> int:
            return hash(self.name)

    # Наследование
    class Dog(Animal):
        def __init__(self, name: str, breed: str) -> None:
            super().__init__(name)
            self.breed = breed

        def speak(self) -> str:
            return f"{self.name} says Woof!"

    # Множественное наследование + MRO
    class Flyable:
        def fly(self) -> str: return "Flying!"

    class FlyingDog(Dog, Flyable): pass
    fd = FlyingDog("Astro", "SuperBreed")
    fd.fly()                                   # "Flying!"
    FlyingDog.__mro__                          # порядок поиска методов

    # dataclass (Python 3.7+)
    from dataclasses import dataclass, field, KW_ONLY

    @dataclass(order=True, frozen=False)
    class Point:
        x: float
        y: float
        _: KW_ONLY
        label: str = ""
        tags: list[str] = field(default_factory=list)

    p1 = Point(1.0, 2.0, label="origin")
    p2 = Point(1.0, 2.0, label="other")
    p1 == p2                                   # True
    ```

---

## Структуры и Value Types

=== "Swift 6"
    ```swift
    // struct = value type (копируется при присваивании)
    struct Point {
        var x: Double
        var y: Double

        // Memberwise initializer генерируется автоматически: Point(x:y:)

        mutating func translate(dx: Double, dy: Double) {
            x += dx; y += dy
        }

        static func origin() -> Point { Point(x: 0, y: 0) }
    }

    var p1 = Point(x: 1, y: 2)
    var p2 = p1                                // КОПИЯ, не ссылка!
    p2.x = 100
    print(p1.x)                                // 1.0 — не изменился!

    // Copy-on-Write (COW): коллекции Swift реально копируются только при изменении
    var arr1 = [1, 2, 3]
    var arr2 = arr1                            // пока нет реальной копии
    arr2.append(4)                            // вот здесь происходит копирование!
    ```

=== "Kotlin 2.x"
    ```kotlin
    // нет отдельного struct, используем data class или value class

    // value class — оборачивает один примитив, нет boxing-оверхеда в runtime
    @JvmInline
    value class UserId(val id: String) {
        init { require(id.isNotBlank()) { "UserId must not be blank" } }
        fun toDisplayString() = "User#$id"
    }

    val uid = UserId("user_123")
    // В JVM bytecode UserId компилируется как обычный String
    // Нет аллокации объекта — pure performance!
    ```

=== "Python 3.13"
    ```python
    # value-семантика через NamedTuple или frozen dataclass

    from typing import NamedTuple

    class Point(NamedTuple):
        x: float
        y: float

    p = Point(1.0, 2.0)
    p.x                                        # 1.0
    p[0]                                       # 1.0 — работает как tuple
    a, b = p                                   # распаковка
    p._asdict()                                # {'x': 1.0, 'y': 2.0}
    p._replace(x=5.0)                          # Point(x=5.0, y=2.0) — новый экземпляр

    # __slots__ — экономия памяти (нет __dict__ на каждом экземпляре)
    @dataclass
    class OptimizedNode:
        __slots__ = ('value', 'next')
        value: int
        next: 'OptimizedNode | None' = None
    ```

---

## Протоколы / Интерфейсы / ABC

=== "Swift 6"
    ```swift
    protocol Drawable {
        var color: String { get set }
        func draw() -> String
        func area() -> Double
    }

    // Реализация по умолчанию через extension
    extension Drawable {
        func describe() -> String {
            "\(color) shape, area: \(String(format: "%.2f", area()))"
        }
    }

    // Protocol с associatedtype (generics для протоколов)
    protocol Repository {
        associatedtype Entity
        associatedtype ID
        func findById(_ id: ID) async throws -> Entity?
        func save(_ entity: Entity) async throws
    }

    // Structural conformance
    struct Circle: Drawable {
        var color: String
        var radius: Double
        func draw() -> String { "◯ Circle" }
        func area() -> Double { .pi * radius * radius }
    }

    // some — opaque type (один конкретный тип, скрытый)
    func makeShape() -> some Drawable { Circle(color: "red", radius: 5) }
    // any — existential type (любой Drawable, boxing оверхед)
    func drawAll(_ shapes: [any Drawable]) { shapes.forEach { print($0.draw()) } }

    // Композиция протоколов
    typealias DrawableAndEquatable = Drawable & Equatable
    ```

=== "Kotlin 2.x"
    ```kotlin
    interface Drawable {
        val color: String
        fun draw(): String
        fun area(): Double
        fun describe() = "$color shape, area: ${"%.2f".format(area())}"  // default
    }

    // Sealed interface — ограниченная иерархия
    sealed interface Shape {
        data class Circle(val radius: Double) : Shape
        data class Rectangle(val w: Double, val h: Double) : Shape
        data class Triangle(val base: Double, val height: Double) : Shape
        data object Unknown : Shape
    }

    // when exhaustive без else!
    fun describeShape(shape: Shape): String = when (shape) {
        is Shape.Circle    -> "Circle r=${shape.radius}"
        is Shape.Rectangle -> "Rect ${shape.w}x${shape.h}"
        is Shape.Triangle  -> "Triangle ${shape.base}x${shape.height}"
        Shape.Unknown      -> "Unknown"
    }

    // Delegation — реализация интерфейса через другой объект (by)
    class LoggingDrawable(private val d: Drawable) : Drawable by d {
        override fun draw(): String {
            println("Drawing...")
            return d.draw()
        }
    }

    // Функциональный интерфейс (SAM)
    fun interface Validator<T> {
        fun validate(value: T): Boolean
    }
    val notEmpty = Validator<String> { it.isNotBlank() }
    ```

=== "Python 3.13"
    ```python
    # Способ 1: ABC (наследование)
    from abc import ABC, abstractmethod

    class Drawable(ABC):
        @property
        @abstractmethod
        def color(self) -> str: ...

        @abstractmethod
        def draw(self) -> str: ...

        @abstractmethod
        def area(self) -> float: ...

        def describe(self) -> str:
            return f"{self.color} shape, area: {self.area():.2f}"

    # Способ 2: Protocol — structural subtyping (РЕКОМЕНДУЕТСЯ в 2026)
    from typing import Protocol, runtime_checkable

    @runtime_checkable
    class DrawableProtocol(Protocol):
        color: str
        def draw(self) -> str: ...
        def area(self) -> float: ...

    # Класс НЕ наследует протокол, но автоматически соответствует ему
    class Circle:
        def __init__(self, color: str, radius: float):
            self.color = color
            self.radius = radius

        def draw(self) -> str: return "◯ Circle"
        def area(self) -> float: return 3.14159 * self.radius ** 2

    c = Circle("red", 5.0)
    isinstance(c, DrawableProtocol)            # True (runtime_checkable)

    def draw_all(shapes: list[DrawableProtocol]) -> None:
        for shape in shapes:
            print(shape.draw())
    ```
