# 📦 Сериализация

=== "Swift 6"
    ```swift
    import Foundation

    // Codable (Encodable + Decodable) — из коробки
    struct User: Codable {
        let id: Int
        let name: String
        let email: String
        let createdAt: Date
    }

    // Кастомные ключи через CodingKeys
    struct Article: Codable {
        let id: Int
        let title: String
        let authorName: String     // в JSON: "author_name"
        let tags: [String]

        enum CodingKeys: String, CodingKey {
            case id, title, tags
            case authorName = "author_name"
        }
    }

    // JSONEncoder / JSONDecoder
    let encoder = JSONEncoder()
    encoder.outputFormatting = [.prettyPrinted, .sortedKeys]
    encoder.dateEncodingStrategy = .iso8601
    encoder.keyEncodingStrategy = .convertToSnakeCase

    let decoder = JSONDecoder()
    decoder.dateDecodingStrategy = .iso8601
    decoder.keyDecodingStrategy = .convertFromSnakeCase

    // Encode → Data
    let user = User(id: 1, name: "Alice", email: "alice@example.com", createdAt: .now)
    let data = try encoder.encode(user)
    let json = String(data: data, encoding: .utf8)!

    // Decode ← Data
    let decoded = try decoder.decode(User.self, from: data)

    // Вложенные generic контейнеры
    struct Response<T: Codable>: Codable {
        let data: T
        let meta: Meta
        struct Meta: Codable { let total: Int; let page: Int }
    }
    let response = try decoder.decode(Response<[User]>.self, from: data)
    ```

=== "Kotlin 2.x"
    ```kotlin
    // kotlinx.serialization (multiplatform)
    import kotlinx.serialization.*
    import kotlinx.serialization.json.*

    @Serializable
    data class User(val id: Int, val name: String, val email: String)

    @Serializable
    data class Article(
        val id: Int,
        val title: String,
        @SerialName("author_name") val authorName: String,
        val tags: List<String> = emptyList(),
        @Transient val cached: Boolean = false     // не сериализуется
    )

    val json = Json {
        prettyPrint = true
        ignoreUnknownKeys = true
        isLenient = true
        encodeDefaults = false
        explicitNulls = false
        namingStrategy = JsonNamingStrategy.SnakeCase
    }

    // Encode / Decode
    val user = User(1, "Alice", "alice@example.com")
    val jsonStr: String = json.encodeToString(user)
    val decoded: User = json.decodeFromString(jsonStr)
    val users: List<User> = json.decodeFromString(jsonList)

    // Polymorphic serialization (sealed class)
    @Serializable
    sealed class Shape {
        @Serializable @SerialName("circle")
        data class Circle(val radius: Double) : Shape()
        @Serializable @SerialName("rect")
        data class Rectangle(val w: Double, val h: Double) : Shape()
    }

    val shape: Shape = Shape.Circle(5.0)
    val shapeJson = json.encodeToString(shape)
    // {"type":"circle","radius":5.0}

    // JsonElement API — работа с динамическим JSON
    val element: JsonElement = json.parseToJsonElement(jsonStr)
    val name = element.jsonObject["name"]?.jsonPrimitive?.content
    ```

=== "Python 3.13"
    ```python
    # Pydantic v2 (стандарт де-факто 2026)
    from pydantic import (BaseModel, Field, field_validator,
                          model_validator, computed_field, ConfigDict)
    from datetime import datetime

    class User(BaseModel):
        id: int
        name: str
        email: str
        created_at: datetime = Field(default_factory=datetime.utcnow)

        model_config = ConfigDict(
            frozen=False,
            str_strip_whitespace=True,
            validate_assignment=True,
            populate_by_name=True,
        )

    class Article(BaseModel):
        id: int
        title: str = Field(min_length=3, max_length=200)
        author_name: str = Field(alias="authorName")
        tags: list[str] = []
        views: int = Field(default=0, ge=0)

        @field_validator("title")
        @classmethod
        def title_must_not_be_empty(cls, v: str) -> str:
            if not v.strip():
                raise ValueError("Title must not be empty")
            return v.strip()

        @computed_field
        @property
        def title_upper(self) -> str:
            return self.title.upper()

    # Создание
    user = User(id=1, name="  Alice  ", email="alice@example.com")
    user.name                                      # "Alice" (strip!)

    # Из JSON строки
    user2 = User.model_validate_json('{"id": 2, "name": "Bob", "email": "b@e.com"}')

    # Сериализация
    user.model_dump()                              # dict
    user.model_dump_json()                         # JSON строка
    user.model_dump(exclude={"created_at"})
    user.model_dump(include={"id", "name"})

    # Обновление (immutable-style)
    updated = user.model_copy(update={"name": "Alice Smith"})

    # Вложенные модели и Generic
    class PaginatedResponse[T](BaseModel):         # Python 3.12+
        data: list[T]
        total: int
        page: int

    response = PaginatedResponse[User](data=[user], total=100, page=1)

    # Discriminated union
    from typing import Literal

    class Circle(BaseModel):
        type: Literal["circle"] = "circle"
        radius: float

    class Rectangle(BaseModel):
        type: Literal["rectangle"] = "rectangle"
        width: float; height: float

    class ShapeWrapper(BaseModel):
        shape: Circle | Rectangle = Field(discriminator="type")
    ```
