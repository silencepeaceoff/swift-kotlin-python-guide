# 🔗 Утечки памяти и циклические ссылки

=== "Swift 6"
    ```swift
    // ARC (Automatic Reference Counting) — детерминированное управление памятью
    // Проблема: циклические ссылки → объекты никогда не освободятся

    class Person {
        let name: String
        var pet: Pet?

        init(name: String) { self.name = name }
        deinit { print("\(name) deinit") }
    }

    class Pet {
        let name: String
        // 🐛 strong → цикл: Person → Pet → Person → ...
        // weak var owner: Person?         // ← РЕШЕНИЕ

        init(name: String) { self.name = name }
        deinit { print("\(name) deinit") }
    }

    // weak — обнуляется при освобождении (только var, только Optional)
    class Apartment {
        weak var tenant: Person?
    }

    // unowned — не обнуляется (краш если объект уже освобождён!)
    class Customer {
        let name: String
        var card: CreditCard?
        init(name: String) { self.name = name }
    }
    class CreditCard {
        unowned let customer: Customer     // Customer всегда живёт дольше
        init(customer: Customer) { self.customer = customer }
    }

    // [weak self] в замыканиях — ОБЯЗАТЕЛЬНО для долгоживущих клоужеров
    class ViewModel {
        var data: [String] = []

        func startLoading() {
            NetworkClient.fetch(url) { [weak self] result in
                guard let self else { return }   // Swift 5.7+ shorthand
                self.data = result
            }
        }

        // Альтернатива: [unowned self] если гарантируем, что self жив
        func loadData() {
            Task { [weak self] in
                let data = try await fetchData()
                self?.data = data
            }
        }
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // JVM Garbage Collector — автоматически обнаруживает циклы
    // Циклические ссылки НЕ проблема для GC (в отличие от ARC)

    // Но утечки всё равно возможны через:
    // 1. Inner class (нестатический) держит ссылку на outer
    class MyActivity {
        inner class InnerTask { /* держит ссылку на MyActivity */ }
        // Решение: class InnerTask (без inner), или companion object
    }

    // 2. Lambda захватывает this
    class Presenter(private val view: View) {
        fun start() {
            networkCall { result ->
                // this@Presenter и view захвачены!
                view.show(result)
            }
        }
    }

    // 3. Listeners / callbacks не удалены
    class MyFragment : Fragment() {
        override fun onStart() {
            EventBus.register(this)
        }
        override fun onStop() {
            EventBus.unregister(this) // ОБЯЗАТЕЛЬНО!
        }
    }

    // WeakReference — аналог weak в Swift
    import java.lang.ref.WeakReference

    class Cache<T>(obj: T) {
        private val ref = WeakReference(obj)
        fun get(): T? = ref.get()           // null если GC собрал
    }
    ```

=== "Python 3.13"
    ```python
    # CPython: reference counting + cyclic GC
    # Reference counting не обрабатывает циклы → cyclic GC дочищает

    import gc
    import weakref

    class Node:
        def __init__(self, name: str) -> None:
            self.name = name
            self.next: "Node | None" = None

        def __del__(self) -> None:
            print(f"{self.name} deleted")

    # Цикл: a → b → a
    a = Node("A")
    b = Node("B")
    a.next = b
    b.next = a
    del a, b
    gc.collect()                               # cyclic GC найдёт и удалит

    # weakref — слабая ссылка
    class Cache:
        def __init__(self) -> None:
            self._data: weakref.WeakValueDictionary[str, object] = \
                weakref.WeakValueDictionary()

        def set(self, key: str, value: object) -> None:
            self._data[key] = value

        def get(self, key: str) -> object | None:
            return self._data.get(key)

    # __slots__ — экономия памяти + предотвращение произвольных атрибутов
    class Point:
        __slots__ = ("x", "y")

        def __init__(self, x: float, y: float) -> None:
            self.x = x
            self.y = y

    # gc модуль — управление циклическим GC
    gc.get_count()                             # пороги поколений
    gc.collect()                               # принудительный запуск
    gc.disable()                               # отключить (для performance-critical кода)
    gc.set_threshold(700, 10, 10)
    ```
