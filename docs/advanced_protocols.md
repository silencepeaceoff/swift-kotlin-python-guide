# 🧬 Протоколы/Интерфейсы: продвинутые паттерны

=== "Swift 6"
    ```swift
    // Conditional conformance — протокол только при определённых условиях
    extension Array: Drawable where Element: Drawable {
        func draw() -> String { map { $0.draw() }.joined(separator: "\n") }
    }

    // Protocol composition
    typealias DrawableAndSaveable = Drawable & Saveable
    func process(_ obj: DrawableAndSaveable) { ... }

    // Existential any (Swift 5.7+)
    var shapes: [any Drawable] = [Circle(...), Square(...)]  // boxing overhead
    // vs. some — статически один тип:
    func makeCircle() -> some Drawable { Circle(color: "red", radius: 5) }

    // Retroactive conformance
    extension URL: Identifiable {
        public var id: String { absoluteString }
    }

    // Protocol with static requirements
    protocol Factory {
        associatedtype Product
        static func make() -> Product
    }

    // Sendable (Swift 6 — строгий concurrency)
    struct MyData: Sendable {
        let id: Int
        let name: String
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Delegation pattern через by
    interface Logger { fun log(msg: String) }
    class ConsoleLogger : Logger { override fun log(msg: String) = println(msg) }
    class TimestampLogger(private val delegate: Logger) : Logger by delegate {
        override fun log(msg: String) =
            delegate.log("[${System.currentTimeMillis()}] $msg")
    }

    // Generic interface с variance
    interface Transformer<in Input, out Output> {
        fun transform(input: Input): Output
    }

    // Context parameters (Kotlin 2.2+)
    context(logger: Logger)
    fun processData(data: List<Int>): Int {
        logger.log("Processing ${data.size} items")
        return data.sum()
    }

    // Functional interfaces (SAM)
    fun interface Predicate<T> { fun test(value: T): Boolean }
    val isEven = Predicate<Int> { it % 2 == 0 }

    // Explicit Backing Fields (Kotlin 2.3.0)
    class ViewModel {
        val state: StateFlow<UiState>
            field = MutableStateFlow(UiState.Loading)
        // Снаружи — StateFlow (read-only), внутри — MutableStateFlow
    }
    ```

=== "Python 3.13"
    ```python
    from typing import Protocol, runtime_checkable, ClassVar

    # Protocol с методами класса и свойствами
    @runtime_checkable
    class Serializable(Protocol):
        @classmethod
        def from_dict(cls, data: dict) -> "Serializable": ...
        def to_dict(self) -> dict: ...
        schema_version: ClassVar[int]

    # Mixin — добавить функциональность через множественное наследование
    class JsonMixin:
        def to_json(self) -> str:
            import json
            return json.dumps(self.to_dict(), default=str, indent=2)

        @classmethod
        def from_json(cls, s: str):
            import json
            return cls.from_dict(json.loads(s))

    class LoggingMixin:
        def log(self, msg: str) -> None:
            import logging
            logging.getLogger(self.__class__.__name__).info(msg)

    # __init_subclass__ — хук при создании подкласса
    class Registry:
        _registry: ClassVar[dict[str, type]] = {}

        def __init_subclass__(cls, name: str | None = None, **kwargs):
            super().__init_subclass__(**kwargs)
            if name:
                Registry._registry[name] = cls

    class Dog(Registry, name="dog"): pass
    class Cat(Registry, name="cat"): pass
    Registry._registry["dog"]                     # <class 'Dog'>
    ```
