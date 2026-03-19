# 📦 Пакетные менеджеры и сборка

=== "Swift 6"
    ```swift
    // Swift Package Manager (SPM)
    // Конфигурация: Package.swift

    // swift-tools-version: 6.0
    import PackageDescription

    let package = Package(
        name: "MyApp",
        platforms: [.macOS(.v14), .iOS(.v17)],
        products: [
            .executable(name: "MyApp", targets: ["MyApp"]),
            .library(name: "MyLib", targets: ["MyLib"]),
        ],
        dependencies: [
            .package(url: "https://github.com/apple/swift-argument-parser", from: "1.3.0"),
            .package(url: "https://github.com/vapor/vapor", from: "4.89.0"),
        ],
        targets: [
            .executableTarget(name: "MyApp", dependencies: [
                "MyLib",
                .product(name: "ArgumentParser", package: "swift-argument-parser"),
            ]),
            .target(name: "MyLib"),
            .testTarget(name: "MyLibTests", dependencies: ["MyLib"]),
        ]
    )

    // Команды:
    // swift build                        — сборка
    // swift run                          — запуск
    // swift test                         — тесты
    // swift package resolve              — обновить зависимости
    // swift package update               — обновить до новых версий
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Gradle Kotlin DSL
    // build.gradle.kts

    plugins {
        kotlin("jvm") version "2.2.0"
        kotlin("plugin.serialization") version "2.2.0"
        application
    }

    group = "com.example"
    version = "1.0.0"

    repositories {
        mavenCentral()
        google()
    }

    dependencies {
        implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
        implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
        implementation("io.ktor:ktor-client-cio:3.0.0")

        testImplementation(kotlin("test"))
        testImplementation("org.junit.jupiter:junit-jupiter:5.10.2")
        testImplementation("io.mockk:mockk:1.13.10")
    }

    application {
        mainClass.set("com.example.MainKt")
    }

    tasks.test { useJUnitPlatform() }

    kotlin { jvmToolchain(21) }

    // Команды:
    // ./gradlew build           — сборка
    // ./gradlew run             — запуск
    // ./gradlew test            — тесты
    // ./gradlew dependencies    — дерево зависимостей
    ```

=== "Python 3.13"
    ```python
    # uv — молниеносный менеджер (рекомендуется в 2026)
    # curl -LsSf https://astral.sh/uv/install.sh | sh

    # Создание проекта
    # uv init myproject
    # cd myproject

    # pyproject.toml (единый конфиг PEP 621)
    """
    [project]
    name = "myproject"
    version = "1.0.0"
    requires-python = ">=3.13"
    dependencies = [
        "httpx>=0.27",
        "pydantic>=2.0",
        "sqlalchemy[asyncio]>=2.0",
    ]

    [project.optional-dependencies]
    dev = [
        "pytest>=8.0",
        "pytest-asyncio>=0.23",
        "mypy>=1.10",
        "ruff>=0.4",
    ]

    [tool.ruff]
    target-version = "py313"
    line-length = 100

    [tool.pytest.ini_options]
    asyncio_mode = "auto"

    [tool.mypy]
    strict = true
    python_version = "3.13"
    """

    # Команды uv:
    # uv sync                    — установить зависимости
    # uv add httpx               — добавить пакет
    # uv add --dev pytest        — добавить dev-зависимость
    # uv remove httpx            — удалить
    # uv run python main.py      — запуск
    # uv run pytest              — тесты
    # uv lock                    — обновить lock-файл
    # uv venv                    — создать virtualenv
    ```
