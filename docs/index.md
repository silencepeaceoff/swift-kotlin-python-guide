# 🗺️ Мега Cheat-Sheet: Swift · Kotlin · Python

**Версии:** Swift 6+, Kotlin 2.2+, Python 3.13+ | **Актуально:** 2026

---

Добро пожаловать в **ультимативный справочник разработчика** — сравнение трёх популярных языков бок о бок. Используйте **вкладки с кодом** на каждой странице, чтобы мгновенно переключаться между Swift, Kotlin и Python.

!!! tip "Как пользоваться"
    На каждой странице блоки кода сгруппированы во вкладки: **Swift 6**, **Kotlin 2.x** и **Python 3.13**. Кликните на нужный язык — и код переключится на месте.

## 📖 Содержание

| Раздел | Описание |
|--------|----------|
| [Переменные, Числа, Строки](basics.md) | Типы данных, память, интерполяция, regex |
| [Коллекции](collections.md) | Arrays, Dicts, Sets — CRUD и функциональные операции |
| [Управление потоком](control_flow.md) | if/switch/when/match, циклы, comprehensions |
| [Функции](functions.md) | Замыкания, декораторы, генераторы |
| [Операторы](operators.md) | Арифметика, перегрузка, scope functions |
| [ООП](oop.md) | Классы, структуры, протоколы/интерфейсы |
| [Enums](enums.md) | Enum, sealed class, Flag |
| [Generics](generics.md) | Обобщения, variance, reified |
| [Async/Await](async.md) | Structured concurrency, Flow, asyncio |
| [Многопоточность](concurrency.md) | GCD, Coroutines, threading/multiprocessing |
| [Обработка ошибок](error_handling.md) | throws, Result, ExceptionGroup |
| [Работа с сетью](networking.md) | URLSession, Ktor, httpx/FastAPI |
| [Работа с файлами](files.md) | FileManager, NIO, pathlib |
| [Базы данных](databases.md) | SwiftData, Room/Exposed, SQLAlchemy |
| [Сериализация](serialization.md) | Codable, kotlinx.serialization, Pydantic |
| [Тесты](testing.md) | Swift Testing, JUnit 5 + MockK, pytest |
| [Отладка](debugging.md) | Логирование, profilers, debuggers |
| [Функциональное программирование](functional.md) | map/filter/reduce, lazy, currying |
| [Result/DSL Builders](dsl_builders.md) | @resultBuilder, DSL, method chaining |
| [Env и CLI](env_cli.md) | Переменные окружения, argparse, Typer |
| [Паттерны проектирования](design_patterns.md) | DI, Repository, Observer |
| [Swift Macros](swift_macros.md) | @Observable, @Model, custom macros |
| [Kotlin Multiplatform](kmp.md) | KMP, expect/actual, Compose |

## 📊 Итоговая таблица: ключевые различия

| Тема | Swift 6 | Kotlin 2.x | Python 3.13 |
|------|---------|------------|-------------|
| Типизация | Статическая, строгая | Статическая, строгая | Динамическая (опц. аннотации) |
| Null-safety | Compile-time `Optional` | Compile-time `?` | Runtime `None` |
| Value types | `struct` (стек/COW) | `@JvmInline value class` | `NamedTuple`, `frozen dataclass` |
| Память | ARC (детерминированный) | JVM GC (G1/ZGC) | Ref-counting + cyclic GC |
| Async | `async/await` + `Actor` | Coroutines + `Flow` | `asyncio` + `async/await` |
| Многопоточность | GCD + Structured concurrency | Coroutines + Dispatchers | threading / multiprocessing / asyncio |
| Ошибки | `throws` / `Result` / Typed throws | `try-catch` / `runCatching` | `try-except` / `ExceptionGroup` |
| Пакеты | SPM (`Package.swift`) | Gradle Kotlin DSL | `uv` + `pyproject.toml` |
| Тесты | Swift Testing / XCTest | JUnit 5 + MockK | pytest + unittest.mock |
| Даты | `Date` + Foundation / Clock API | `java.time` / kotlinx-datetime | `datetime` + `zoneinfo` |
| Сеть | URLSession (async) | Ktor Client | httpx / aiohttp |
| Файлы | FileManager + URL | File + NIO Path | `pathlib.Path` |
| Access control | `private`/`fileprivate`/`internal`/`package`/`public`/`open` | `private`/`protected`/`internal`/`public` | Соглашения `_` и `__` |
