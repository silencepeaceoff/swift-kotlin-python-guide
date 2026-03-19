# 🔮 Generics

=== "Swift 6"
    ```swift
    // Обобщённая функция с constraint
    func maxValue<T: Comparable>(_ a: T, _ b: T) -> T { a > b ? a : b }
    maxValue(3, 7)                             // 7
    maxValue("apple", "banana")               // "banana"

    // Обобщённый тип
    struct Stack<Element> {
        private var items: [Element] = []

        mutating func push(_ item: Element) { items.append(item) }
        mutating func pop() -> Element? { items.popLast() }
        var top: Element? { items.last }
        var isEmpty: Bool { items.isEmpty }
        var count: Int { items.count }
    }

    // Extension с constraint
    extension Stack where Element: Equatable {
        func contains(_ element: Element) -> Bool { items.contains(element) }
    }

    // Несколько constraints через where
    func merge<T>(_ a: T, _ b: T) -> T where T: Equatable & Comparable { a < b ? b : a }

    // Primary associated types (Swift 5.7+)
    // Использование: some Sequence<Int>, any Sequence<String>

    // some / any (Swift 5.7+)
    func makeCollection() -> some Collection { [1, 2, 3] }   // opaque
    func process(_ col: any Collection) { print(col.count) }  // existential
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Обобщённая функция
    fun <T : Comparable<T>> maxValue(a: T, b: T): T = if (a > b) a else b

    // Несколько bounds
    fun <T> process(value: T) where T : Comparable<T>, T : Cloneable { ... }

    // Обобщённый класс
    class Stack<T> {
        private val items = ArrayDeque<T>()

        fun push(item: T) = items.addLast(item)
        fun pop(): T? = if (items.isEmpty()) null else items.removeLast()
        val top: T? get() = items.lastOrNull()
        val isEmpty: Boolean get() = items.isEmpty()
    }

    // Variance — IN/OUT (ключевое отличие от Swift/Python)
    // out T = covariant: только возвращает T (Producer)
    interface Producer<out T> { fun produce(): T }
    // in T = contravariant: только принимает T (Consumer)
    interface Consumer<in T> { fun consume(item: T) }

    // List<out T> — поэтому List<Dog> совместим с List<Animal>

    // Use-site variance
    fun copy(from: Array<out Any>, to: Array<Any>) { ... }

    // Reified — доступ к типу T в runtime (ТОЛЬКО в inline fun)
    inline fun <reified T> isInstanceOf(value: Any): Boolean = value is T
    inline fun <reified T> parseJson(json: String): T = Json.decodeFromString<T>(json)

    isInstanceOf<String>("hello")            // true — без Class<T> параметра!
    ```

=== "Python 3.13"
    ```python
    from typing import TypeVar, Generic

    T = TypeVar("T")
    T_co = TypeVar("T_co", covariant=True)

    # Обобщённый класс
    class Stack(Generic[T]):
        def __init__(self) -> None:
            self._items: list[T] = []

        def push(self, item: T) -> None: self._items.append(item)
        def pop(self) -> T | None: return self._items.pop() if self._items else None

        @property
        def top(self) -> T | None: return self._items[-1] if self._items else None

    stack: Stack[int] = Stack()
    stack.push(1); stack.push(2)

    # TypeVar с bounds
    Numeric = TypeVar("Numeric", int, float, complex)

    def add(a: Numeric, b: Numeric) -> Numeric:
        return a + b

    # Python 3.12+ — новый синтаксис (PEP 695, чище и лаконичнее)
    def max_val[T: (int, float, str)](a: T, b: T) -> T:
        return a if a > b else b

    class Box[T]:
        def __init__(self, value: T) -> None:
            self.value = value

        def map[U](self, func: "Callable[[T], U]") -> "Box[U]":
            return Box(func(self.value))

    # Protocol с generics
    from typing import Protocol

    class Comparable[T](Protocol):
        def __lt__(self, other: T) -> bool: ...
        def __gt__(self, other: T) -> bool: ...
    ```
