# iOS Development Conventions

This document defines iOS/Swift specific coding standards. For general commands and workflow, see `README.md`.

---

## Project Structure

### Standard iOS Project Layout
```
ProjectName/
├── App/                    # App entry point, AppDelegate, SceneDelegate
├── Core/                   # Core business logic, models
├── Features/              # Feature modules
│   ├── FeatureName/
│   │   ├── Views/
│   │   ├── ViewModels/
│   │   └── Services/
├── Common/                # Shared components
│   ├── Extensions/
│   ├── Utilities/
│   └── UI/
├── Resources/             # Assets, colors, fonts
└── Configuration/         # Config files, plist
```

---

## Naming Conventions

### Files
- Swift files: PascalCase (e.g., `UserProfileViewController.swift`)
- Test files: append `Tests` (e.g., `UserProfileViewControllerTests.swift`)
- Extensions: `TypeName+Extension.swift` (e.g., `String+Validation.swift`)

### Code
- Classes/Structs/Enums: PascalCase
- Variables/Functions: camelCase
- Constants: camelCase (not SCREAMING_SNAKE_CASE)
- Protocols: Descriptive noun or `-able`/`-ible` suffix (e.g., `Downloadable`)

---

## Swift Coding Style

### Prefer Modern Swift
```swift
// Good: Use modern async/await
func fetchUser() async throws -> User {
    try await apiClient.fetch("/user")
}

// Avoid: Completion handlers when async/await is available
func fetchUser(completion: @escaping (Result<User, Error>) -> Void) { }
```

### Optionals
```swift
// Good: Use guard for early returns
guard let user else { return }

// Good: Use if let for scoped unwrapping
if let email = user.email {
    sendEmail(to: email)
}

// Avoid: Force unwrapping unless absolutely safe
let user = userStore.currentUser! // Avoid
```

### Type Inference
```swift
// Good: Let Swift infer when obvious
let name = "John"
let users = [User]()

// Good: Be explicit when needed for clarity
let timeout: TimeInterval = 30
```

---

## Access Control

**Principle: Use the most restrictive access level that fulfills the requirement.**

### Access Levels (most to least restrictive)
- `private` - Within declaration and extensions in same file
- `fileprivate` - Within entire file
- `internal` - Within module (default)
- `public` - Outside module, cannot subclass/override
- `open` - Outside module, can subclass/override

### Best Practices

```swift
// Properties should be private by default
class ViewModel {
    private var users: [User] = []
    public var userCount: Int { users.count }
}

// Use private(set) for read-only public properties
class DataManager {
    public private(set) var isLoading: Bool = false
}

// Singleton with private initializer
public class ServiceManager {
    public static let shared = ServiceManager()
    private init() {}
}
```

---

## Architecture Patterns

### Preferred: MVVM with Combine/SwiftUI
- Use `@StateObject` for view models in SwiftUI
- Use `@Published` for observable properties
- Keep views thin, move logic to ViewModels

### UIKit Projects
- Use MVVM or VIP (Clean Architecture)
- Avoid massive view controllers
- Separate concerns: View, ViewModel/Presenter, Service

---

## Dependency Injection

```swift
// Good: Protocol-based DI
protocol UserServiceProtocol {
    func fetchUser() async throws -> User
}

class UserViewModel {
    private let userService: UserServiceProtocol

    init(userService: UserServiceProtocol) {
        self.userService = userService
    }
}

// Avoid: Direct instantiation of dependencies
class UserViewModel {
    private let userService = UserService() // Hard to test
}
```

---

## Error Handling

```swift
// Good: Custom error types
enum NetworkError: LocalizedError {
    case invalidURL
    case noConnection
    case unauthorized

    var errorDescription: String? {
        switch self {
        case .invalidURL: return "Invalid URL"
        case .noConnection: return "No internet connection"
        case .unauthorized: return "Unauthorized access"
        }
    }
}

// Good: Propagate errors with throws
func fetchData() async throws -> Data {
    try await network.request()
}
```

---

## Testing Standards

### Unit Tests
- Write tests for business logic
- Use dependency injection for testability
- Mock external dependencies
- Aim for >70% code coverage on critical paths

### Test Naming
```swift
func test_fetchUser_whenSuccessful_returnsUser() {
    // given
    // when
    // then
}
```

---

## Common iOS Patterns

### Singleton Pattern
```swift
class NetworkManager {
    static let shared = NetworkManager()
    private init() {}
}
```

### Delegate Pattern
```swift
protocol DataProviderDelegate: AnyObject {
    func didReceiveData(_ data: Data)
}

class DataProvider {
    weak var delegate: DataProviderDelegate?
}
```

### Completion Handler Pattern
```swift
typealias CompletionHandler = (Result<Data, Error>) -> Void

func fetchData(completion: @escaping CompletionHandler) {
    // Implementation
}
```

---

## Performance Considerations

- Use `lazy var` for expensive computations
- Avoid doing heavy work on main thread
- Use `@MainActor` for UI updates
- Profile with Instruments before optimizing
- Consider memory impact of closures and captures

---

## Security

- Never commit API keys or secrets
- Use Keychain for sensitive data
- Validate all user input
- Use HTTPS for network calls
- Enable App Transport Security (ATS)

---

## Localization

- All user-facing strings should be localized
- Use `NSLocalizedString` or SwiftGen
- Support right-to-left languages when needed

---

## Accessibility

- Add accessibility labels to UI elements
- Support Dynamic Type
- Test with VoiceOver
- Ensure sufficient color contrast

---

## Documentation

### Code Comments
```swift
// Good: Explain WHY, not WHAT
// Delayed dispatch prevents animation conflicts
DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
    self.updateUI()
}

// Avoid: Stating the obvious
// Set the title
title = "Home"
```

### Public APIs
```swift
/// Fetches user profile from the remote server
/// - Parameter userID: The unique identifier of the user
/// - Returns: A User object containing profile information
/// - Throws: NetworkError if the request fails
func fetchUserProfile(userID: String) async throws -> User {
    // Implementation
}
```

---

## Git Workflow

### Branch Naming
- Feature: `feature/description` or `feature/developer-name/ticket-number`
- Bug fix: `fix/description`
- Hotfix: `hotfix/description`

### Commit Messages
- Use conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`
- Be descriptive: `feat: add user authentication flow`
- Reference tickets when applicable: `fix: resolve crash on launch (#123)`

---

## Xcode Settings

- Enable strict concurrency checking
- Use SwiftLint for consistent style
- Enable "Treat Warnings as Errors" for release builds
- Use xcconfig files for build configurations

---

*Version: 2.0 | Updated: 2026-01-20*
