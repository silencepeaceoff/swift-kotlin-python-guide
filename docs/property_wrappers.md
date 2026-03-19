# 🏷️ Property Wrappers / Delegates / Descriptors

=== "Swift 6"
    ```swift
    // Создание собственного wrapper
    @propertyWrapper
    struct Clamped<T: Comparable> {
        private var value: T
        let range: ClosedRange<T>

        var wrappedValue: T {
            get { value }
            set { value = min(max(newValue, range.lowerBound), range.upperBound) }
        }

        var projectedValue: ClosedRange<T> { range }

        init(wrappedValue: T, _ range: ClosedRange<T>) {
            self.range = range
            self.value = min(max(wrappedValue, range.lowerBound), range.upperBound)
        }
    }

    struct Player {
        @Clamped(0...100) var health: Int = 100
        @Clamped(0...10)  var speed: Double = 5.0
    }

    var p = Player()
    p.health = 150          // автоматически 100
    p.health = -10          // автоматически 0
    print(p.$health)        // projectedValue: 0...100

    // Встроенные wrappers SwiftUI / Swift:
    // @Published       — уведомляет подписчиков (Combine)
    // @State           — локальное состояние View
    // @Binding         — двусторонняя привязка
    // @Environment     — значение из окружения SwiftUI
    // @AppStorage      — UserDefaults
    // @Observable      — Swift 5.9+ макрос

    // lazy — инициализация при первом обращении
    class DataManager {
        lazy var heavyData: [Int] = {
            print("Loading...")
            return Array(0..<10_000)
        }()
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    import kotlin.properties.Delegates
    import kotlin.properties.ReadWriteProperty
    import kotlin.reflect.KProperty

    // Встроенные делегаты
    class Example {
        val data: List<Int> by lazy {
            println("Computing...")
            (1..1000).toList()
        }

        var name: String by Delegates.observable("Initial") { prop, old, new ->
            println("${prop.name}: $old → $new")
        }

        var age: Int by Delegates.vetoable(0) { _, _, new ->
            new >= 0                               // отклонить отрицательные
        }

        var token: String by Delegates.notNull()
    }

    // Своя реализация делегата
    class Clamped<T : Comparable<T>>(
        private var value: T,
        private val range: ClosedRange<T>
    ) : ReadWriteProperty<Any?, T> {
        override fun getValue(thisRef: Any?, property: KProperty<*>): T = value
        override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
            this.value = value.coerceIn(range)
        }
    }

    fun <T : Comparable<T>> clamped(initial: T, range: ClosedRange<T>) =
        Clamped(initial, range)

    class Player {
        var health: Int by clamped(100, 0..100)
        var speed: Double by clamped(5.0, 0.0..10.0)
    }

    // Делегирование свойств к Map
    class Config(private val map: Map<String, Any?>) {
        val host: String by map
        val port: Int by map
        val debug: Boolean by map
    }
    val config = Config(mapOf("host" to "localhost", "port" to 8080, "debug" to false))
    ```

=== "Python 3.13"
    ```python
    # Descriptors (__get__ / __set__ / __delete__)
    class Clamped:
        """Дескриптор: зажимает значение в диапазон [min_val, max_val]"""
        def __set_name__(self, owner, name: str) -> None:
            self.name = name
            self.private_name = f"_{name}"

        def __init__(self, min_val, max_val):
            self.min_val = min_val
            self.max_val = max_val

        def __get__(self, obj, objtype=None):
            if obj is None: return self
            return getattr(obj, self.private_name, self.min_val)

        def __set__(self, obj, value):
            clamped = max(self.min_val, min(self.max_val, value))
            setattr(obj, self.private_name, clamped)

    class Player:
        health: int = Clamped(0, 100)
        speed: float = Clamped(0.0, 10.0)

    p = Player()
    p.health = 150          # автоматически 100
    p.health = -10          # автоматически 0

    # @property — удобный синтаксис для дескрипторов
    class Temperature:
        def __init__(self, celsius: float = 0.0) -> None:
            self._celsius = celsius

        @property
        def celsius(self) -> float:
            return self._celsius

        @celsius.setter
        def celsius(self, value: float) -> None:
            if value < -273.15:
                raise ValueError("Below absolute zero!")
            self._celsius = value

        @property
        def fahrenheit(self) -> float:
            return self._celsius * 9/5 + 32

    # cached_property (Python 3.8+)
    from functools import cached_property

    class DataProcessor:
        def __init__(self, data: list[int]) -> None:
            self.data = data

        @cached_property
        def statistics(self) -> dict:
            print("Computing stats...")
            return {
                "mean": sum(self.data) / len(self.data),
                "max":  max(self.data),
                "min":  min(self.data),
            }

    dp = DataProcessor([1, 2, 3, 4, 5])
    dp.statistics                                 # "Computing stats..."
    dp.statistics                                 # из кэша
    del dp.statistics                             # сбросить кэш
    ```
