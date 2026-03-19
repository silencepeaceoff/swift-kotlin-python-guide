# 📁 Работа с файлами

=== "Swift 6"
    ```swift
    import Foundation

    let fm = FileManager.default
    let docs = fm.urls(for: .documentDirectory, in: .userDomainMask)[0]
    let fileURL = docs.appendingPathComponent("data.txt")

    // Запись / чтение текста
    try "Hello, File!".write(to: fileURL, atomically: true, encoding: .utf8)
    let content = try String(contentsOf: fileURL, encoding: .utf8)

    // Запись / чтение Data
    let data = Data([0x48, 0x65, 0x6C, 0x6C, 0x6F])
    try data.write(to: fileURL)
    let readData = try Data(contentsOf: fileURL)

    // JSON Codable
    struct Config: Codable { let host: String; let port: Int }

    let config = Config(host: "localhost", port: 8080)
    let encoder = JSONEncoder()
    encoder.outputFormatting = [.prettyPrinted, .sortedKeys]
    try encoder.encode(config).write(to: fileURL)

    let loaded = try JSONDecoder().decode(Config.self, from: Data(contentsOf: fileURL))

    // Файловые операции
    fm.fileExists(atPath: fileURL.path)
    try fm.copyItem(at: fileURL, to: docs.appendingPathComponent("backup.txt"))
    try fm.moveItem(at: fileURL, to: docs.appendingPathComponent("moved.txt"))
    try fm.removeItem(at: fileURL)
    try fm.createDirectory(at: docs.appendingPathComponent("subdir"),
                           withIntermediateDirectories: true)

    // Список содержимого директории
    let contents = try fm.contentsOfDirectory(
        at: docs,
        includingPropertiesForKeys: [.fileSizeKey, .creationDateKey],
        options: .skipsHiddenFiles
    )
    for url in contents {
        let attrs = try url.resourceValues(forKeys: [.fileSizeKey])
        print("\(url.lastPathComponent): \(attrs.fileSize ?? 0) bytes")
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    import java.io.File
    import java.nio.file.*
    import kotlinx.serialization.*
    import kotlinx.serialization.json.*

    // Запись / чтение текста
    File("data.txt").writeText("Hello, File!", Charsets.UTF_8)
    File("data.txt").appendText("\nAppended line")
    val content = File("data.txt").readText()
    val lines: List<String> = File("data.txt").readLines()

    // Построчное чтение (эффективно для больших файлов)
    File("big.txt").forEachLine { line -> process(line) }
    File("big.txt").bufferedReader().use { reader ->
        reader.lineSequence().forEach { process(it) }
    }

    // Байты
    File("image.png").readBytes()
    File("output.bin").writeBytes(byteArray)

    // JSON — kotlinx.serialization
    @Serializable
    data class Config(val host: String, val port: Int)

    val config = Config("localhost", 8080)
    val json   = Json { prettyPrint = true }
    File("config.json").writeText(json.encodeToString(config))
    val loaded = json.decodeFromString<Config>(File("config.json").readText())

    // NIO Path API (рекомендуется для новых проектов)
    val path = Path("data") / "config.json"
    path.parent.createDirectories()
    path.writeText("content")

    path.exists()
    path.isRegularFile(); path.isDirectory()
    path.fileSize()
    path.deleteIfExists()
    path.moveTo(Path("new_location.json"))
    Files.copy(path, Path("backup.json"), StandardCopyOption.REPLACE_EXISTING)

    // Список файлов
    Files.list(Path(".")).use { stream ->
        stream.filter { it.isRegularFile() }.forEach { println(it) }
    }
    Files.walk(Path(".")).use { stream ->
        stream.filter { it.extension == "kt" }.forEach { println(it) }
    }
    ```

=== "Python 3.13"
    ```python
    from pathlib import Path
    import json, csv, tomllib

    # Path — объектно-ориентированный путь
    base = Path("data")
    p = base / "config.json"

    # Создание директорий
    p.parent.mkdir(parents=True, exist_ok=True)

    # Запись / чтение текста
    p.write_text("Hello!", encoding="utf-8")
    content = p.read_text(encoding="utf-8")

    # Байты
    Path("img.png").write_bytes(b"\x89PNG...")
    raw = Path("img.png").read_bytes()

    # Свойства пути
    p.name                         # "config.json"
    p.stem                         # "config"
    p.suffix                       # ".json"
    p.parent                       # Path("data")
    p.absolute()                   # абсолютный путь
    p.resolve()                    # абсолютный + разрешение symlinks

    # Файловые операции
    p.exists(); p.is_file(); p.is_dir()
    p.unlink(missing_ok=True)
    p.rename(base / "new_name.json")
    import shutil
    shutil.copy(p, base / "backup.json")
    shutil.rmtree(base)            # удалить директорию рекурсивно

    # Список файлов
    for f in Path(".").iterdir(): print(f)
    list(Path(".").glob("*.py"))               # все .py
    list(Path(".").rglob("*.py"))              # рекурсивно

    # JSON
    with p.open("w", encoding="utf-8") as f:
        json.dump({"host": "localhost", "port": 8080}, f, indent=2)

    with p.open(encoding="utf-8") as f:
        config = json.load(f)

    # CSV
    csv_path = Path("data.csv")
    with csv_path.open("w", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=["name", "age"])
        writer.writeheader()
        writer.writerows([{"name": "Alice", "age": 30}, {"name": "Bob", "age": 25}])

    # TOML (встроен с Python 3.11)
    with open("pyproject.toml", "rb") as f:
        config = tomllib.load(f)

    # Построчное чтение большого файла
    with open("big_file.txt", encoding="utf-8") as f:
        for line in f:                         # O(1) память
            process(line.rstrip("\n"))

    # Временный файл
    import tempfile
    with tempfile.NamedTemporaryFile(mode="w", suffix=".txt", delete=False) as f:
        f.write("temp content")
        temp_path = Path(f.name)
    temp_path.unlink()
    ```
