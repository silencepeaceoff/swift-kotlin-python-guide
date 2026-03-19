# 🌍 Переменные окружения и CLI аргументы

=== "Swift 6"
    ```swift
    import Foundation

    // Переменные окружения
    let env = ProcessInfo.processInfo.environment
    let dbUrl = env["DATABASE_URL"] ?? "sqlite://local.db"
    let debug = env["DEBUG"].map { $0 == "1" || $0.lowercased() == "true" } ?? false

    // Аргументы командной строки
    let args = CommandLine.arguments

    // Парсинг через swift-argument-parser (рекомендуется)
    import ArgumentParser

    @main
    struct MyApp: ParsableCommand {
        static var configuration = CommandConfiguration(
            commandName: "myapp",
            abstract: "My command-line app"
        )

        @Option(name: .shortAndLong, help: "Port number")
        var port: Int = 8080

        @Flag(name: .shortAndLong, help: "Enable debug mode")
        var debug = false

        @Argument(help: "Input file path")
        var input: String

        mutating func run() throws {
            print("Running on port \(port), debug: \(debug), input: \(input)")
        }
    }
    // ./myapp --port 9090 --debug myfile.txt
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Переменные окружения
    val dbUrl = System.getenv("DATABASE_URL") ?: "sqlite://local.db"
    val debug = System.getenv("DEBUG")?.let { it == "1" || it == "true" } ?: false

    // Все переменные окружения
    val allEnv: Map<String, String> = System.getenv()

    // Системные свойства (java -DmyProp=value)
    val myProp = System.getProperty("myProp", "default")

    // Аргументы из main
    fun main(args: Array<String>) {
        println("Args: ${args.toList()}")
    }

    // Clikt (популярная библиотека для Kotlin CLI)
    import com.github.ajalt.clikt.core.CliktCommand
    import com.github.ajalt.clikt.parameters.options.*
    import com.github.ajalt.clikt.parameters.arguments.*

    class MyApp : CliktCommand(help = "My CLI app") {
        val port: Int by option("--port", "-p", help = "Port number").int().default(8080)
        val debug: Boolean by option("--debug", "-d").flag()
        val input: String by argument(help = "Input file")

        override fun run() {
            echo("Running on port $port, debug: $debug, input: $input")
        }
    }

    fun main(args: Array<String>) = MyApp().main(args)
    ```

=== "Python 3.13"
    ```python
    import os
    import sys

    # Переменные окружения
    db_url = os.environ.get("DATABASE_URL", "sqlite:///local.db")
    debug  = os.environ.get("DEBUG", "false").lower() in ("1", "true", "yes")

    # Установить переменную окружения
    os.environ["MY_VAR"] = "value"

    # .env файлы (pip install python-dotenv)
    from dotenv import load_dotenv
    load_dotenv()                                  # загружает .env файл

    # argparse (встроен в stdlib)
    import argparse

    parser = argparse.ArgumentParser(description="My CLI app")
    parser.add_argument("--port", "-p", type=int, default=8080, help="Port number")
    parser.add_argument("--debug", "-d", action="store_true", help="Debug mode")
    parser.add_argument("input", help="Input file path")

    args = parser.parse_args()
    print(f"Port: {args.port}, Debug: {args.debug}, Input: {args.input}")

    # Typer — современный CLI с type hints (рекомендуется)
    import typer
    from typing import Annotated

    app = typer.Typer()

    @app.command()
    def main(
        input: Annotated[str, typer.Argument(help="Input file")],
        port: Annotated[int, typer.Option("--port", "-p", help="Port")] = 8080,
        debug: Annotated[bool, typer.Option("--debug", "-d")] = False,
    ) -> None:
        """My awesome CLI app."""
        typer.echo(f"Running on port {port}, debug: {debug}, input: {input}")

    if __name__ == "__main__":
        app()
    ```
