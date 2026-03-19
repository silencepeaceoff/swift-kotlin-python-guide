# 🧪 Тесты

=== "Swift 6"
    ```swift
    // Swift Testing (новый стандарт Swift 6+) + XCTest

    // ── Swift Testing ──
    import Testing

    @Suite("UserService Tests")
    struct UserServiceTests {

        @Test("Creates user with valid data")
        func userCreation() async throws {
            let service = UserService(repo: InMemoryRepo())
            let user = try await service.create(name: "Alice", age: 30)

            #expect(user.name == "Alice")
            #expect(user.age == 30)
            #expect(user.id != nil)
        }

        // Параметризованные тесты
        @Test("Rejects invalid age", arguments: [-1, 0, 151, 200])
        func invalidAge(age: Int) async throws {
            let service = UserService(repo: InMemoryRepo())
            await #expect(throws: ValidationError.self) {
                try await service.create(name: "Bob", age: age)
            }
        }
    }

    // ── XCTest — классический подход ──
    import XCTest

    class CalculatorTests: XCTestCase {
        var sut: Calculator!

        override func setUp() async throws {
            sut = Calculator()
        }
        override func tearDown() async throws {
            sut = nil
        }

        func testAddition() {
            XCTAssertEqual(sut.add(2, 3), 5)
            XCTAssertNotNil(sut)
            XCTAssertTrue(sut.isReady)
        }

        func testAsyncOp() async throws {
            let result = try await sut.fetchResult()
            XCTAssertNotNil(result)
        }

        func testPerformance() throws {
            measure { _ = heavyOperation() }
        }
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    // JUnit 5 + MockK + kotlinx-coroutines-test

    import org.junit.jupiter.api.*
    import org.junit.jupiter.params.ParameterizedTest
    import org.junit.jupiter.params.provider.ValueSource
    import io.mockk.*
    import kotlinx.coroutines.test.*
    import app.cash.turbine.*
    import kotlin.test.*

    @TestInstance(TestInstance.Lifecycle.PER_CLASS)
    class UserServiceTest {

        private lateinit var mockRepo: UserRepository
        private lateinit var service: UserService

        @BeforeEach
        fun setUp() {
            mockRepo = mockk()
            service = UserService(mockRepo)
        }

        @AfterEach
        fun tearDown() { clearAllMocks() }

        @Test
        fun `should return user when found`() = runTest {
            // Arrange
            val expected = User(1, "Alice")
            coEvery { mockRepo.findById(1) } returns expected

            // Act
            val result = service.getUser(1)

            // Assert
            assertEquals(expected, result)
            coVerify { mockRepo.findById(1) }
        }

        @ParameterizedTest
        @ValueSource(ints = [-1, 0, 151])
        fun `should throw for invalid age`(age: Int) = runTest {
            assertThrows<IllegalArgumentException> {
                service.createUser("Bob", age)
            }
        }

        // Тест Flow с Turbine
        @Test
        fun `should emit loading then success`() = runTest {
            coEvery { mockRepo.findAll() } returns listOf(User(1, "Alice"))

            service.usersFlow.test {
                assertEquals(UiState.Loading, awaitItem())
                assertEquals(UiState.Success(listOf(User(1, "Alice"))), awaitItem())
                awaitComplete()
            }
        }
    }
    ```

=== "Python 3.13"
    ```python
    # pytest (абсолютный стандарт 2026) + unittest.mock
    import pytest
    from unittest.mock import AsyncMock, MagicMock, patch
    from dataclasses import dataclass

    # ── Fixtures — setup/teardown через dependency injection ──
    @pytest.fixture
    def mock_repo() -> AsyncMock:
        return AsyncMock()

    @pytest.fixture
    def service(mock_repo: AsyncMock) -> "UserService":
        return UserService(repo=mock_repo)

    # ── Базовый тест ──
    @dataclass
    class User:
        id: int
        name: str

    class UserService:
        def __init__(self, repo): self.repo = repo
        async def get_user(self, uid: int) -> User: return await self.repo.find_by_id(uid)
        async def create_user(self, name: str, age: int) -> User:
            if not (0 < age < 150): raise ValueError(f"Invalid age: {age}")
            return await self.repo.save(User(id=0, name=name))

    @pytest.mark.asyncio
    async def test_get_user_success(service: UserService, mock_repo: AsyncMock):
        mock_repo.find_by_id.return_value = User(id=1, name="Alice")
        result = await service.get_user(1)
        assert result.name == "Alice"
        mock_repo.find_by_id.assert_awaited_once_with(1)

    # ── Параметризованные тесты ──
    @pytest.mark.parametrize("age", [-1, 0, 150, 200])
    @pytest.mark.asyncio
    async def test_invalid_age_raises(service: UserService, age: int):
        with pytest.raises(ValueError, match="Invalid age"):
            await service.create_user("Bob", age)

    # ── patch — подменить зависимость ──
    @pytest.mark.asyncio
    async def test_with_patch():
        with patch("mymodule.external_api", new_callable=AsyncMock) as mock_api:
            mock_api.return_value = {"status": "ok"}
            result = await my_function()
            assert result["status"] == "ok"

    # ── Coverage ──
    # pytest --cov=mypackage --cov-report=html
    ```
