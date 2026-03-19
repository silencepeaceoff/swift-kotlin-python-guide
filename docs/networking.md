# 🌐 Работа с сетью

=== "Swift 6"
    ```swift
    // URLSession (нативный, async/await)
    import Foundation

    struct Post: Codable {
        let id: Int; let title: String; let body: String; let userId: Int
    }

    // GET
    func fetchPost(id: Int) async throws -> Post {
        let url = URL(string: "https://jsonplaceholder.typicode.com/posts/\(id)")!
        let (data, response) = try await URLSession.shared.data(from: url)
        guard let http = response as? HTTPURLResponse,
              (200..<300).contains(http.statusCode) else {
            throw URLError(.badServerResponse)
        }
        return try JSONDecoder().decode(Post.self, from: data)
    }

    // POST с телом
    func createPost(_ post: Post) async throws -> Post {
        var request = URLRequest(url: URL(string: "https://jsonplaceholder.typicode.com/posts")!)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        request.httpBody = try JSONEncoder().encode(post)

        let (data, _) = try await URLSession.shared.data(for: request)
        return try JSONDecoder().decode(Post.self, from: data)
    }

    // Кастомная URLSession
    let config = URLSessionConfiguration.default
    config.timeoutIntervalForRequest = 30
    config.httpAdditionalHeaders = ["Accept": "application/json"]
    let session = URLSession(configuration: config)

    // WebSocket
    let wsTask = URLSession.shared.webSocketTask(with: URL(string: "wss://echo.example.com")!)
    wsTask.resume()
    try await wsTask.send(.string("Hello!"))
    let message = try await wsTask.receive()
    if case .string(let text) = message { print(text) }
    wsTask.cancel(with: .normalClosure, reason: nil)
    ```

=== "Kotlin 2.x"
    ```kotlin
    // Ktor Client (coroutine-based, multiplatform)
    import io.ktor.client.*
    import io.ktor.client.call.*
    import io.ktor.client.engine.cio.*
    import io.ktor.client.plugins.contentnegotiation.*
    import io.ktor.client.request.*
    import io.ktor.serialization.kotlinx.json.*
    import kotlinx.serialization.Serializable

    @Serializable
    data class Post(val id: Int, val title: String, val body: String, val userId: Int)

    val client = HttpClient(CIO) {
        install(ContentNegotiation) { json() }
        install(HttpTimeout) {
            requestTimeoutMillis = 30_000
            connectTimeoutMillis = 10_000
        }
        defaultRequest {
            header("Authorization", "Bearer $token")
            header("Accept", "application/json")
        }
    }

    // GET
    suspend fun fetchPost(id: Int): Post =
        client.get("https://jsonplaceholder.typicode.com/posts/$id").body()

    // POST
    suspend fun createPost(post: Post): Post =
        client.post("https://jsonplaceholder.typicode.com/posts") {
            contentType(ContentType.Application.Json)
            setBody(post)
        }.body()

    // DELETE
    suspend fun deletePost(id: Int) {
        val response = client.delete("https://jsonplaceholder.typicode.com/posts/$id")
        check(response.status.isSuccess()) { "Delete failed: ${response.status}" }
    }

    // WebSocket
    client.webSocket("wss://echo.example.com") {
        send(Frame.Text("Hello!"))
        val frame = incoming.receive() as? Frame.Text
        println(frame?.readText())
    }

    client.close()
    ```

=== "Python 3.13"
    ```python
    # httpx (sync + async, современный стандарт)
    import httpx
    from pydantic import BaseModel

    class Post(BaseModel):
        id: int | None = None
        title: str
        body: str
        user_id: int

    # Синхронный (для скриптов)
    with httpx.Client(
        base_url="https://jsonplaceholder.typicode.com",
        timeout=30.0,
        headers={"Authorization": f"Bearer {token}"}
    ) as client:
        response = client.get("/posts/1")
        response.raise_for_status()
        post = Post(**response.json())

    # Асинхронный (для приложений — РЕКОМЕНДУЕТСЯ)
    async def fetch_post(post_id: int) -> Post:
        async with httpx.AsyncClient(
            base_url="https://jsonplaceholder.typicode.com",
            timeout=httpx.Timeout(connect=5.0, read=30.0)
        ) as client:
            r = await client.get(f"/posts/{post_id}")
            r.raise_for_status()
            return Post(**r.json())

    async def create_post(post: Post) -> Post:
        async with httpx.AsyncClient() as client:
            r = await client.post(
                "https://jsonplaceholder.typicode.com/posts",
                json=post.model_dump(exclude={"id"})
            )
            r.raise_for_status()
            return Post(**r.json())

    # WebSocket
    import websockets

    async def ws_chat():
        async with websockets.connect("wss://echo.example.com") as ws:
            await ws.send("Hello!")
            reply = await ws.recv()
            print(reply)

    # FastAPI — создание REST API
    from fastapi import FastAPI, HTTPException

    app = FastAPI()

    @app.get("/posts/{post_id}", response_model=Post)
    async def get_post(post_id: int) -> Post:
        try:
            return await fetch_post(post_id)
        except httpx.HTTPStatusError as e:
            raise HTTPException(status_code=e.response.status_code, detail=str(e))

    @app.post("/posts", response_model=Post, status_code=201)
    async def add_post(post: Post) -> Post:
        return await create_post(post)
    ```
