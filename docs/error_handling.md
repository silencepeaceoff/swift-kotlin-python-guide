# ⚠️ Обработка ошибок

=== "Swift 6"
    ```swift
    enum AppError: Error, LocalizedError {
        case notFound(id: String)
        case networkError(underlying: Error)
        case validationFailed(fields: [String])

        var errorDescription: String? {
            switch self {
            case .notFound(let id):    return "Resource \(id) not found"
            case .networkError(let e): return "Network: \(e.localizedDescription)"
            case .validationFailed(let f): return "Invalid fields: \(f.joined(separator: ", "))"
            }
        }
    }

    // throws — обычная ошибка
    func findUser(id: String) throws -> User {
        guard !id.isEmpty else { throw AppError.notFound(id: id) }
        return User(id: id, name: "Alice")
    }

    // do-catch
    do {
        let user = try findUser(id: "42")
        print(user.name)
    } catch AppError.notFound(let id) {
        print("Not found: \(id)")
    } catch let error as AppError {
        print("App error: \(error.errorDescription ?? "")")
    } catch {
        print("Unknown: \(error)")             // error всегда доступен
    }

    // try? — превращает в Optional
    let user = try? findUser(id: "42")        // User? — nil при ошибке

    // try! — краш при ошибке
    let user2 = try! findUser(id: "42")

    // Result<T, E> — явный возврат ошибки без throws
    func fetchUser(id: String) -> Result<User, AppError> {
        guard !id.isEmpty else { return .failure(.notFound(id: id)) }
        return .success(User(id: id, name: "Alice"))
    }

    let result = fetchUser(id: "42")
    let name = result.map { $0.name }         // Result<String, AppError>

    // Typed throws (Swift 6 — строгая типизация ошибок)
    func strictFetch(id: String) throws(AppError) -> User {
        throw .notFound(id: id)              // ТОЛЬКО AppError, не любой Error
    }
    do { try strictFetch(id: "x") } catch { print(error.errorDescription ?? "") }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // все исключения unchecked (нет checked exceptions как в Java)
    class AppException(
        val code: Int,
        message: String,
        cause: Throwable? = null
    ) : Exception(message, cause)

    class NotFoundException(val id: String) : AppException(404, "Not found: $id")

    fun findUser(id: String): User {
        if (id.isBlank()) throw NotFoundException(id)
        return User(id, "Alice")
    }

    // try-catch-finally
    try {
        val user = findUser("42")
        println(user.name)
    } catch (e: NotFoundException) {
        println("Not found: ${e.id}")
    } catch (e: AppException) {
        println("App error ${e.code}: ${e.message}")
    } catch (e: Exception) {
        println("Unknown: ${e.message}")
    } finally {
        println("Always")
    }

    // try как выражение
    val user = try { findUser("42") } catch (e: Exception) { null }

    // Result (Kotlin stdlib)
    val result = runCatching { findUser("42") }
    result.onSuccess { println(it.name) }
    result.onFailure { println(it.message) }
    result.getOrDefault(User("default", "Unknown"))
    result.getOrElse { e -> User("error", e.message ?: "?") }
    result.map { it.name }                    // Result<String>
    result.recover { User("fallback", "Fallback") }

    // Полезные assertions
    fun process(value: String?) {
        val v = requireNotNull(value) { "value must not be null" }
        require(v.isNotEmpty()) { "value must not be empty" }
        check(v.length < 100) { "value too long" }
    }
    ```

=== "Python 3.13"
    ```python
    class AppError(Exception):
        """Базовый класс ошибок приложения"""
        def __init__(self, message: str, code: int = 0) -> None:
            super().__init__(message)
            self.code = code

    class NotFoundError(AppError):
        def __init__(self, resource_id: str) -> None:
            self.resource_id = resource_id
            super().__init__(f"Resource {resource_id!r} not found", code=404)

    class ValidationError(AppError):
        def __init__(self, fields: list[str]) -> None:
            self.fields = fields
            super().__init__(f"Invalid fields: {', '.join(fields)}", code=422)

    # try-except-else-finally
    def find_user(user_id: str) -> dict:
        if not user_id:
            raise NotFoundError(user_id)
        return {"id": user_id, "name": "Alice"}

    try:
        user = find_user("42")
    except NotFoundError as e:
        print(f"Not found: {e.resource_id}")
    except (ValidationError, AppError) as e:
        print(f"App error {e.code}: {e}")
    except Exception as e:
        print(f"Unknown: {e}")
    else:
        print(f"Success: {user['name']}")     # только если не было исключения
    finally:
        print("Always runs")

    # Exception Groups (Python 3.11+) — одновременно несколько исключений
    def validate_all(data: dict) -> None:
        errors = []
        if not data.get("name"): errors.append(ValueError("name required"))
        if not data.get("email"): errors.append(ValueError("email required"))
        if errors: raise ExceptionGroup("Validation failed", errors)

    try:
        validate_all({})
    except* ValueError as eg:
        for e in eg.exceptions:
            print(f"  - {e}")

    # add_note (Python 3.11+)
    try:
        1 / 0
    except ZeroDivisionError as e:
        e.add_note("This happened while processing user input")
        raise

    # contextlib утилиты
    from contextlib import suppress, contextmanager

    with suppress(FileNotFoundError, PermissionError):
        open("missing.txt").read()
    ```
