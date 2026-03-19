# 🛡️ Модификаторы доступа

| Модификатор | Swift 6 | Kotlin 2.x | Python |
|-------------|---------|------------|--------|
| **Самый закрытый** | `private` — только в объявлении | `private` — только в классе | `__name` — name mangling |
| **Файл** | `fileprivate` — файл | — | `_name` — соглашение |
| **Модуль/пакет** | `internal` (по умолчанию) — модуль | `internal` (по умолчанию) — модуль | Нет аналога |
| **Пакет** | `package` (Swift 5.9+) — package | — | — |
| **Публичный** | `public` — виден, нельзя override | `public` — виден | Всё по умолчанию |
| **Открытый** | `open` — можно наследовать/override | `open` class — можно наследовать | Всё по умолчанию |
| **Подклассы** | — | `protected` — класс + наследники | — |

=== "Swift 6"
    ```swift
    // private — только внутри {} где объявлен
    class Account {
        private var balance: Double = 0
        private(set) var ownerName: String   // public read, private write

        func deposit(_ amount: Double) {
            balance += amount                // OK
        }
    }

    // fileprivate — доступно всем типам в этом файле
    fileprivate func helper() { }

    // internal (по умолчанию) — доступно внутри модуля
    class MyService { }                    // == internal class MyService

    // package (Swift 5.9+) — доступно внутри пакета
    package func packageOnlyFunc() { }

    // public — виден снаружи модуля, нельзя наследовать/переопределить
    public struct PublicData { }

    // open — можно наследовать/переопределить вне модуля
    open class BaseVC { open func setup() { } }
    ```

=== "Kotlin 2.x"
    ```kotlin
    class Account {
        private var balance: Double = 0.0
        protected open fun audit() { }     // подклассы видят
    }

    // Kotlin unique: internal = модуль (не пакет!)
    internal class ModuleOnlyService { }

    // Kotlin по умолчанию final (нельзя наследовать без open)
    open class Base {
        open fun method() { }
    }
    class Derived : Base() {
        final override fun method() { }    // финальная переопредёлённая версия
    }

    // sealed — ограниченная иерархия в том же файле/модуле
    sealed interface Shape { }
    ```

=== "Python 3.13"
    ```python
    class Account:
        def __init__(self) -> None:
            self._balance = 0.0        # "protected" по соглашению
            self.__secret = "hidden"   # name mangling → _Account__secret

    # _ = "не трогай, это internal"
    # __ = "серьёзно не трогай, name mangling"
    # __slots__ = ограничить набор атрибутов

    acc = Account()
    acc._balance                         # работает (соглашение, не запрет)
    acc._Account__secret                 # доступ через mangled имя

    # __all__ — экспорт при from module import *
    __all__ = ["PublicClass", "public_func"]
    ```
