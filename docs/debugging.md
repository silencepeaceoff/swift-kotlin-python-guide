# 🐛 Отладка (Debugging)

=== "Swift 6"
    ```swift
    // print / dump
    let user = User(id: 1, name: "Alice")
    print(user)                                    // User(id: 1, name: "Alice")
    dump(user)                                     // структурированный вывод

    // assert — только в Debug, в Release выключается
    assert(!arr.isEmpty, "Array must not be empty")
    assertionFailure("This line must never be reached")

    // precondition — работает и в Release
    precondition(index >= 0, "Index must be non-negative")

    // fatalError — всегда крашит
    func notImplemented() -> Never {
        fatalError("Not implemented yet")
    }

    // #if DEBUG — компиляция только в Debug
    #if DEBUG
    print("Debug info: \(user)")
    #endif

    // Логирование через os.Logger
    import os
    let logger = Logger(subsystem: "com.myapp", category: "network")
    logger.debug("Request started: \(url, privacy: .public)")
    logger.error("Failed: \(error.localizedDescription)")
    logger.info("User loaded: \(user.name, privacy: .private)")

    // lldb команды:
    // po user           — print object
    // p user.name       — print expression
    // bt                — backtrace
    // frame variable    — все переменные
    ```

=== "Kotlin 2.x"
    ```kotlin
    // println / toString
    val user = User(1, "Alice")
    println(user)                                  // data class: User(id=1, name=Alice)

    // require / check / assert
    require(user.name.isNotBlank()) { "Name must not be blank" }  // IllegalArgumentException
    check(isConnected) { "Must be connected" }                    // IllegalStateException

    // Логирование (slf4j + logback — стандарт в JVM)
    import org.slf4j.LoggerFactory

    val log = LoggerFactory.getLogger(this::class.java)
    log.debug("Request: {}", url)                  // {} — placeholder
    log.info("User loaded: {}", user.name)
    log.error("Failed", exception)                 // автоматически stacktrace

    // Kotlin Logging (удобная обёртка)
    import mu.KotlinLogging
    private val logger = KotlinLogging.logger {}
    logger.debug { "Request: $url" }               // lambda — не вычисляется если disabled!

    // Измерение времени
    val elapsed = measureTimeMillis { heavyWork() }
    println("Took: ${elapsed}ms")

    // Проверка типов
    println(user::class.simpleName)               // "User"
    println(user::class.qualifiedName)            // "com.example.User"
    ```

=== "Python 3.13"
    ```python
    # print с метаданными
    import sys
    print("Debug:", user, file=sys.stderr)

    # pprint — красивый вывод сложных структур
    from pprint import pprint
    pprint({"key": [1, 2, {"nested": True}]}, depth=3, width=80)

    # assert
    assert len(arr) > 0, "Array must not be empty"
    # ВАЖНО: assert отключается при python -O!

    # breakpoint() — встроенный дебаггер (Python 3.7+)
    def complex_function(data):
        result = process(data)
        breakpoint()                               # ← остановка здесь
        return result

    # pdb команды:
    # n (next), s (step), c (continue), p expr, l — код, w — стек, q — quit

    # Логирование (stdlib)
    import logging

    logging.basicConfig(
        level=logging.DEBUG,
        format="%(asctime)s [%(levelname)s] %(name)s: %(message)s",
    )

    logger = logging.getLogger(__name__)
    logger.debug("Request: %s", url)
    logger.info("User loaded: %s", user.name)
    logger.exception("Unexpected error")          # error + traceback

    # Структурированное логирование (structlog — рекомендуется)
    import structlog
    log = structlog.get_logger()
    log.info("user_loaded", user_id=1, name="Alice", duration_ms=42)

    # traceback — детали исключения
    import traceback
    try:
        risky()
    except Exception:
        traceback.print_exc()

    # time.perf_counter — точное измерение
    import time
    start = time.perf_counter()
    heavy_work()
    print(f"Took: {time.perf_counter() - start:.4f}s")

    # cProfile — профилировщик
    import cProfile
    cProfile.run("heavy_function()", sort="cumulative")
    ```
