# 🎨 Паттерны проектирования

## Dependency Injection (DI)

=== "Swift 6"
    ```swift
    // Протокол-ориентированное DI (чистый Swift)
    protocol UserRepository: Sendable {
        func findById(_ id: Int) async throws -> User?
        func save(_ user: User) async throws
    }

    // Реальная реализация
    final class SwiftDataUserRepository: UserRepository {
        private let container: ModelContainer

        init(container: ModelContainer) { self.container = container }

        func findById(_ id: Int) async throws -> User? {
            let context = container.mainContext
            let descriptor = FetchDescriptor<User>(
                predicate: #Predicate { $0.id == id }
            )
            return try context.fetch(descriptor).first
        }

        func save(_ user: User) async throws {
            let context = container.mainContext
            context.insert(user)
            try context.save()
        }
    }

    // Mock для тестов
    actor InMemoryUserRepository: UserRepository {
        private var users: [Int: User] = [:]

        func findById(_ id: Int) async throws -> User? { users[id] }
        func save(_ user: User) async throws { users[user.id] = user }
    }

    // Service зависит от протокола
    @MainActor
    class UserService: ObservableObject {
        private let repo: any UserRepository

        init(repo: any UserRepository) { self.repo = repo }

        func getUser(_ id: Int) async throws -> User {
            guard let user = try await repo.findById(id) else {
                throw AppError.notFound(id: "\(id)")
            }
            return user
        }
    }

    // DI Container паттерн
    @MainActor
    final class AppContainer {
        lazy var modelContainer = try! ModelContainer(for: User.self)
        lazy var userRepo: any UserRepository = SwiftDataUserRepository(container: modelContainer)
        lazy var userService = UserService(repo: userRepo)
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // ── Koin (Lightweight DI, стандарт для Kotlin / KMP) ──
    import org.koin.core.module.dsl.*
    import org.koin.dsl.*

    // Определение зависимостей
    val appModule = module {
        singleOf(::AppDatabase)                    // singleton
        factory { get<AppDatabase>().userDao() }   // factory: новый экземпляр каждый раз

        singleOf(::UserRepositoryImpl) { bind<UserRepository>() }
        viewModelOf(::UserViewModel)               // scoped to ViewModel lifecycle
    }

    // Инициализация (Application.onCreate для Android)
    startKoin { modules(appModule) }

    // Injection в ViewModel
    class UserViewModel(private val repo: UserRepository) : ViewModel() {
        val users: StateFlow<List<User>> = repo.getAllFlow()
            .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())
    }

    // Injection в Activity/Fragment
    class UserFragment : Fragment() {
        private val viewModel: UserViewModel by viewModel()
    }

    // Тест — подмена зависимости
    class UserServiceTest : KoinTest {
        @get:Rule val koinRule = KoinTestRule.create {
            modules(module {
                singleOf(::InMemoryUserRepository) { bind<UserRepository>() }
            })
        }

        val repo: UserRepository by inject()

        @Test fun `should find user`() = runTest {
            repo.save(User(1, "Alice"))
            val user = repo.findById(1)
            assertEquals("Alice", user?.name)
        }
    }
    ```

=== "Python 3.13"
    ```python
    # ── FastAPI Depends (встроенный DI) ──
    from fastapi import FastAPI, Depends
    from sqlalchemy.ext.asyncio import AsyncSession

    app = FastAPI()

    async def get_session() -> AsyncSession:
        async with AsyncSessionLocal() as session:
            yield session                          # cleanup автоматически

    async def get_user_repo(session: AsyncSession = Depends(get_session)):
        return SQLAlchemyUserRepository(session)

    @app.get("/users/{user_id}")
    async def get_user(
        user_id: int,
        repo: UserRepository = Depends(get_user_repo)
    ) -> User:
        user = await repo.find_by_id(user_id)
        if not user:
            raise HTTPException(status_code=404)
        return user

    # ── Pure Python DI (без библиотек) ──
    from abc import ABC, abstractmethod
    from dataclasses import dataclass

    @dataclass
    class User:
        id: int
        name: str
        email: str

    class UserRepository(ABC):
        @abstractmethod
        async def find_by_id(self, uid: int) -> User | None: ...

    class SQLAlchemyUserRepository(UserRepository):
        def __init__(self, session: AsyncSession) -> None:
            self._session = session

        async def find_by_id(self, uid: int) -> User | None:
            return await self._session.get(User, uid)

    class InMemoryUserRepository(UserRepository):
        def __init__(self) -> None:
            self._users: dict[int, User] = {}

        async def find_by_id(self, uid: int) -> User | None:
            return self._users.get(uid)

    class UserService:
        def __init__(self, repo: UserRepository) -> None:
            self._repo = repo

        async def get_user(self, uid: int) -> User:
            user = await self._repo.find_by_id(uid)
            if user is None:
                raise ValueError(f"User {uid} not found")
            return user

    # Production
    service = UserService(repo=SQLAlchemyUserRepository(session))

    # Тест
    async def test_get_user() -> None:
        repo = InMemoryUserRepository()
        repo._users[1] = User(id=1, name="Alice", email="alice@example.com")
        service = UserService(repo=repo)
        user = await service.get_user(1)
        assert user.name == "Alice"
    ```
