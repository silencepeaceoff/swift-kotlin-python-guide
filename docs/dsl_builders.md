# 🏗️ Result Builders / DSL Builders

=== "Swift 6"
    ```swift
    // @resultBuilder — основа SwiftUI

    @resultBuilder
    struct ArrayBuilder<T> {
        static func buildBlock(_ components: T...) -> [T] { components }
        static func buildIf(_ component: [T]?) -> [T] { component ?? [] }
        static func buildEither(first component: [T]) -> [T] { component }
        static func buildEither(second component: [T]) -> [T] { component }
        static func buildArray(_ components: [[T]]) -> [T] { components.flatMap { $0 } }
    }

    func makeArray<T>(@ArrayBuilder<T> _ build: () -> [T]) -> [T] { build() }

    let numbers = makeArray {
        1
        2
        3
        if Bool.random() { 4 }
        for i in 5...7 { i }
    }

    // HTML DSL пример
    @resultBuilder
    struct HTML {
        static func buildBlock(_ parts: String...) -> String {
            parts.joined(separator: "\n")
        }
    }

    func div(@HTML content: () -> String) -> String {
        "<div>\n\(content())\n</div>"
    }
    func p(_ text: String) -> String { "<p>\(text)</p>" }
    func h1(_ text: String) -> String { "<h1>\(text)</h1>" }

    let page = div {
        h1("Hello, World!")
        p("This is a paragraph.")
        p("Another paragraph.")
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // DSL через lambda with receiver + extension functions
    // Основа Gradle KTS, Ktor routing, Jetpack Compose, Exposed

    class HtmlBuilder {
        private val sb = StringBuilder()

        fun h1(text: String) = sb.appendLine("<h1>$text</h1>")
        fun p(text: String)  = sb.appendLine("<p>$text</p>")
        fun div(block: HtmlBuilder.() -> Unit) {
            sb.appendLine("<div>")
            this.block()
            sb.appendLine("</div>")
        }
        fun build() = sb.toString()
    }

    fun html(block: HtmlBuilder.() -> Unit): String =
        HtmlBuilder().apply(block).build()

    val page = html {
        div {
            h1("Hello, World!")
            p("First paragraph")
            p("Second paragraph")
        }
    }

    // Type-safe builder для конфигурации
    data class ServerConfig(
        val host: String = "localhost",
        val port: Int = 8080,
        val debug: Boolean = false,
        val database: DatabaseConfig = DatabaseConfig()
    )

    data class DatabaseConfig(
        val url: String = "jdbc:sqlite:local.db",
        val poolSize: Int = 10
    )

    class ServerConfigBuilder {
        var host = "localhost"
        var port = 8080
        var debug = false
        private var dbConfig = DatabaseConfig()

        fun database(block: DatabaseConfig.() -> DatabaseConfig) {
            dbConfig = DatabaseConfig().block()
        }
        fun build() = ServerConfig(host, port, debug, dbConfig)
    }

    fun server(block: ServerConfigBuilder.() -> Unit): ServerConfig =
        ServerConfigBuilder().apply(block).build()

    val config = server {
        host = "0.0.0.0"
        port = 9090
        debug = true
    }
    ```

=== "Python 3.13"
    ```python
    from contextlib import contextmanager
    from dataclasses import dataclass, field

    # Method chaining builder
    class HtmlBuilder:
        def __init__(self) -> None:
            self._parts: list[str] = []

        def h1(self, text: str) -> "HtmlBuilder":
            self._parts.append(f"<h1>{text}</h1>")
            return self

        def p(self, text: str) -> "HtmlBuilder":
            self._parts.append(f"<p>{text}</p>")
            return self

        def build(self) -> str:
            return "\n".join(self._parts)

    page = (
        HtmlBuilder()
        .h1("Hello, World!")
        .p("First paragraph")
        .p("Second paragraph")
        .build()
    )

    # Query Builder DSL
    @dataclass
    class Query:
        """SQL-подобный Query Builder"""
        table: str
        _conditions: list[str] = field(default_factory=list, repr=False)
        _limit: int | None = field(default=None, repr=False)
        _order_by: str | None = field(default=None, repr=False)

        def where(self, condition: str) -> "Query":
            self._conditions.append(condition)
            return self

        def limit(self, n: int) -> "Query":
            self._limit = n
            return self

        def order_by(self, column: str) -> "Query":
            self._order_by = column
            return self

        def build(self) -> str:
            sql = f"SELECT * FROM {self.table}"
            if self._conditions:
                sql += " WHERE " + " AND ".join(self._conditions)
            if self._order_by:
                sql += f" ORDER BY {self._order_by}"
            if self._limit:
                sql += f" LIMIT {self._limit}"
            return sql

    query = (
        Query("users")
        .where("age > 18")
        .where("active = TRUE")
        .order_by("name")
        .limit(10)
        .build()
    )
    # SELECT * FROM users WHERE age > 18 AND active = TRUE ORDER BY name LIMIT 10
    ```
