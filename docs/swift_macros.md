# 🔩 Swift Macros (Swift 5.9+)

!!! note "Только Swift"
    Макросы — уникальная возможность Swift. Kotlin использует KSP/KotlinPoet для кодогенерации, Python — декораторы и metaclass.

```swift
// ── Встроенные / Apple макросы ──

// @Observable (Swift 5.9+, заменяет ObservableObject + @Published)
import Observation

@Observable
class ViewModel {
    var items: [Item] = []
    var isLoading: Bool = false
    var errorMessage: String? = nil

    func load() async {
        isLoading = true
        defer { isLoading = false }
        items = try! await fetchItems()
    }
}

// В SwiftUI — не нужен @StateObject / @ObservedObject
struct ContentView: View {
    @State private var vm = ViewModel()

    var body: some View {
        if vm.isLoading { ProgressView() }
        else { List(vm.items) { Text($0.name) } }
    }
}

// #Preview (Swift 5.9+)
#Preview {
    ContentView()
}

// #stringify — встроенный expression macro
let (value, string) = #stringify(2 + 3)
// value = 5, string = "2 + 3"

// ── Создание собственного макроса ──
// Требует отдельного Package target с типом .macro

import SwiftSyntax
import SwiftSyntaxMacros

public struct UnwrappedMacro: ExpressionMacro {
    public static func expansion(
        of node: some FreestandingMacroExpansionSyntax,
        in context: some MacroExpansionContext
    ) throws -> ExprSyntax {
        let arg = node.argumentList.first!.expression
        return "\(arg) ?? { fatalError(\"Unexpected nil: \(arg)\") }()"
    }
}

// Объявление макроса в Public API
@freestanding(expression)
public macro Unwrapped<T>(_ value: T?) -> T = #externalMacro(
    module: "MyMacrosImpl",
    type: "UnwrappedMacro"
)

// Использование
let value = #Unwrapped(optionalValue)

// ── Attached macros ──

@attached(extension, conformances: Equatable, names: named(==))
public macro AutoEquatable() = #externalMacro(
    module: "MyMacrosImpl", type: "AutoEquatableMacro"
)

@AutoEquatable
struct Point {
    var x: Double
    var y: Double
    var label: String
    // Сгенерированный код:
    // static func == (lhs: Point, rhs: Point) -> Bool {
    //     lhs.x == rhs.x && lhs.y == rhs.y && lhs.label == rhs.label
    // }
}

// Типы макросов:
// @freestanding(expression) — #macro() в выражениях
// @freestanding(declaration) — #macro() для объявлений
// @attached(member)          — добавляет членов к типу
// @attached(memberAttribute) — добавляет атрибуты к членам
// @attached(accessor)        — добавляет get/set
// @attached(extension)       — добавляет extension с conformance
// @attached(peer)            — генерирует рядом новые объявления
```
