# 🎯 Enums

=== "Swift 6"
    ```swift
    // Самые мощные enum'ы среди трёх языков
    enum Direction { case north, south, east, west }

    // С ассоциированными значениями (алгебраические типы данных)
    enum NetworkResult<T> {
        case success(T)
        case failure(code: Int, message: String)
        case loading(progress: Double)
    }

    // С raw values и методами
    enum HTTPMethod: String, CaseIterable {
        case get = "GET", post = "POST", put = "PUT", delete = "DELETE"

        var requiresBody: Bool {
            switch self { case .post, .put: return true; default: return false }
        }
    }
    HTTPMethod.allCases                        // [.get, .post, .put, .delete]
    HTTPMethod(rawValue: "POST")               // Optional(.post)

    // Pattern matching
    let result: NetworkResult<String> = .success("Hello")
    switch result {
    case .success(let data):                   print("Got: \(data)")
    case .failure(let code, let msg):          print("\(code): \(msg)")
    case .loading(let progress):               print("\(Int(progress*100))%")
    }

    // if case — удобно для одного кейса
    if case .success(let data) = result { print(data) }

    // Рекурсивный enum (для деревьев, AST и т.д.)
    indirect enum Tree<T> {
        case leaf(T)
        case node(Tree<T>, T, Tree<T>)
    }
    ```

=== "Kotlin 2.x"
    ```kotlin
    enum class Direction {
        NORTH, SOUTH, EAST, WEST;

        fun opposite(): Direction = when (this) {
            NORTH -> SOUTH; SOUTH -> NORTH
            EAST  -> WEST;  WEST  -> EAST
        }
    }

    // Enum с свойствами
    enum class Planet(val mass: Double, val radius: Double) {
        MERCURY(3.303e+23, 2.4397e6),
        VENUS(4.869e+24, 6.0518e6),
        EARTH(5.976e+24, 6.37814e6);

        companion object {
            const val G = 6.67300E-11
        }
        val surfaceGravity = G * mass / (radius * radius)
    }

    // sealed class — мощнее enum (полноценные классы, разные поля)
    sealed class ApiResponse<out T> {
        data class Success<T>(val data: T, val timestamp: Long = System.currentTimeMillis()) : ApiResponse<T>()
        data class Error(val code: Int, val message: String) : ApiResponse<Nothing>()
        data object Loading : ApiResponse<Nothing>()
        data object Empty : ApiResponse<Nothing>()
    }

    fun handleResponse(response: ApiResponse<String>) = when (response) {
        is ApiResponse.Success  -> println("Data: ${response.data}")
        is ApiResponse.Error    -> println("${response.code}: ${response.message}")
        ApiResponse.Loading     -> println("Loading...")
        ApiResponse.Empty       -> println("No data")
    }
    ```

=== "Python 3.13"
    ```python
    from enum import Enum, IntEnum, StrEnum, Flag, auto

    class Direction(Enum):
        NORTH = "north"; SOUTH = "south"
        EAST = "east";   WEST = "west"

        def opposite(self) -> "Direction":
            opposites = {
                Direction.NORTH: Direction.SOUTH,
                Direction.SOUTH: Direction.NORTH,
                Direction.EAST: Direction.WEST,
                Direction.WEST: Direction.EAST,
            }
            return opposites[self]

    Direction.NORTH.value                      # "north"
    Direction("north")                         # Direction.NORTH
    list(Direction)                            # все варианты

    # auto() — автоматические значения
    class Color(Enum):
        RED = auto(); GREEN = auto(); BLUE = auto()

    # StrEnum (Python 3.11+) — совместим с str
    class Status(StrEnum):
        PENDING  = auto()                      # "pending"
        ACTIVE   = auto()                      # "active"
        INACTIVE = auto()                      # "inactive"

    Status.ACTIVE == "active"                  # True

    # IntEnum — совместим с int
    class HttpStatus(IntEnum):
        OK = 200; NOT_FOUND = 404; ERROR = 500

    HttpStatus.OK == 200                       # True

    # Flag — для битовых флагов
    class Permission(Flag):
        READ    = auto()
        WRITE   = auto()
        EXECUTE = auto()
        ALL     = READ | WRITE | EXECUTE

    perm = Permission.READ | Permission.WRITE
    Permission.READ in perm                    # True
    ```
