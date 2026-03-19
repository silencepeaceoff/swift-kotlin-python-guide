# 🌐 Kotlin Multiplatform (KMP)

!!! note "Только Kotlin"
    KMP — уникальная возможность Kotlin для кроссплатформенной разработки: один код для iOS, Android, Desktop, Web.

```kotlin
// ── Структура проекта ──
// my-project/
// ├── shared/                    ← общий KMP модуль
// │   ├── src/
// │   │   ├── commonMain/        ← общий код (все платформы)
// │   │   ├── commonTest/
// │   │   ├── androidMain/       ← Android-специфика
// │   │   ├── iosMain/           ← iOS-специфика
// │   │   └── desktopMain/       ← Desktop-специфика
// ├── androidApp/                ← Android приложение
// ├── iosApp/                    ← iOS приложение (Xcode)
// └── desktopApp/                ← Desktop приложение

// shared/build.gradle.kts
kotlin {
    androidTarget { compilations.all { kotlinOptions { jvmTarget = "17" } } }
    iosX64(); iosArm64(); iosSimulatorArm64()
    jvm("desktop")

    sourceSets {
        commonMain.dependencies {
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.9.0")
            implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.7.3")
            implementation("io.ktor:ktor-client-core:3.0.0")
        }
        androidMain.dependencies {
            implementation("io.ktor:ktor-client-android:3.0.0")
        }
        iosMain.dependencies {
            implementation("io.ktor:ktor-client-darwin:3.0.0")
        }
    }
}

// ── expect / actual — платформозависимый код ──

// commonMain: объявляем «контракт»
expect class PlatformInfo() {
    val name: String
    val version: String
}

expect fun currentTimeMillis(): Long

// androidMain: реализация для Android
actual class PlatformInfo actual constructor() {
    actual val name = "Android"
    actual val version = android.os.Build.VERSION.RELEASE
}

actual fun currentTimeMillis(): Long = System.currentTimeMillis()

// iosMain: реализация для iOS
actual class PlatformInfo actual constructor() {
    actual val name = "iOS"
    actual val version = UIDevice.currentDevice.systemVersion
}

actual fun currentTimeMillis(): Long =
    (NSDate().timeIntervalSince1970 * 1000).toLong()

// ── Общий код (commonMain) ──

interface UserRepository {
    suspend fun getUsers(): List<User>
    suspend fun getUserById(id: Int): User?
}

class UserViewModel(private val repo: UserRepository) {
    private val _state = MutableStateFlow<UserState>(UserState.Loading)
    val state: StateFlow<UserState> = _state.asStateFlow()

    fun loadUsers() {
        viewModelScope.launch {
            _state.value = try {
                UserState.Success(repo.getUsers())
            } catch (e: Exception) {
                UserState.Error(e.message ?: "Unknown error")
            }
        }
    }
}

sealed class UserState {
    data object Loading : UserState()
    data class Success(val users: List<User>) : UserState()
    data class Error(val message: String) : UserState()
}
```
