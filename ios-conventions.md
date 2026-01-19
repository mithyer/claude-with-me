# iOS Development Conventions with Claude

This document defines the collaboration standards between developers and Claude for iOS projects.

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

### Access Control

**Principle: Use the most restrictive access level that fulfills the requirement.**

#### Access Levels (from most to least restrictive)
- `private` - Accessible only within the declaration and its extensions in the same file
- `fileprivate` - Accessible within the entire file
- `internal` - Default level, accessible within the module
- `public` - Accessible outside the module, but cannot be subclassed or overridden
- `open` - Accessible outside the module and can be subclassed or overridden

#### Guidelines

```swift
// Good: Explicitly declare access control for clarity
public class UserManager {
    // Private for implementation details
    private let database: Database
    private var cache: [String: User] = [:]

    // Internal for module-wide access (default, but explicit is better)
    internal let configuration: Configuration

    // Public for external API
    public func fetchUser(id: String) async throws -> User {
        // Implementation
    }

    // Private helper method
    private func updateCache(user: User) {
        cache[user.id] = user
    }
}

// Good: Use fileprivate when sharing between types in same file
fileprivate protocol CacheProtocol {
    func cache(_ item: Any)
}

class CacheManager: CacheProtocol {
    fileprivate func cache(_ item: Any) {
        // Implementation
    }
}

// Good: Use open for classes/methods designed for inheritance
open class BaseViewController: UIViewController {
    // open allows subclasses outside the module to override
    open func configure() {
        // Default implementation
    }

    // public prevents override outside the module
    public final func setup() {
        // Cannot be overridden
    }
}
```

#### Best Practices

**1. Properties should be private by default**
```swift
class ViewModel {
    // Good: Private backing storage
    private var users: [User] = []

    // Good: Public computed property for read access
    public var userCount: Int {
        users.count
    }
}
```

**2. Use private(set) for read-only public properties**
```swift
class DataManager {
    // Good: Public read, private write
    public private(set) var isLoading: Bool = false

    public func loadData() {
        isLoading = true
        // Load data
        isLoading = false
    }
}
```

**3. Framework/Library development**
```swift
// For types that should be extensible outside the module
open class NetworkClient {
    // open: Can be overridden by framework users
    open func request(_ endpoint: String) async throws -> Data {
        // Default implementation
    }
}

// For types that should not be subclassed
public final class Logger {
    // public: Accessible but cannot be subclassed
    public func log(_ message: String) {
        // Implementation
    }
}
```

**4. Protocol conformance**
```swift
// Internal protocol for app architecture
protocol ViewModelProtocol {
    func loadData()
}

// Implementation keeps protocol conformance internal
class UserViewModel: ViewModelProtocol {
    // Public facing API
    public init() {}

    // Internal protocol requirement
    func loadData() {
        // Implementation
    }

    // Private helpers
    private func fetchFromNetwork() {}
}
```

**5. Extensions access control**
```swift
// Public extension on public type
public extension String {
    // Public method available to module users
    func trimmed() -> String {
        trimmingCharacters(in: .whitespacesAndNewlines)
    }
}

// Private extension for internal helpers
private extension Array {
    func chunked(into size: Int) -> [[Element]] {
        // Implementation only needed internally
    }
}
```

#### Common Patterns

**Singleton with private initializer**
```swift
public class ServiceManager {
    public static let shared = ServiceManager()

    // Private prevents external instantiation
    private init() {}

    public func performService() {
        // Implementation
    }
}
```

**Nested types inherit access level**
```swift
public class Container {
    // Internal by default, not accessible outside module
    struct Configuration {
        let timeout: TimeInterval
    }

    // Must explicitly make public if needed
    public struct Settings {
        public let apiKey: String
        public init(apiKey: String) {
            self.apiKey = apiKey
        }
    }
}
```

#### Avoid

```swift
// Avoid: Unnecessary fileprivate when private is sufficient
fileprivate class Helper { } // Use private if only used in one type

// Avoid: Making everything public "just in case"
public class InternalViewModel { } // Should be internal

// Avoid: Inconsistent access levels
public class Manager {
    var data: [String] = [] // Should be private or explicit internal
}

// Avoid: Missing access control on init for public types
public struct User {
    let name: String
    // Missing: public init - this struct cannot be instantiated outside module!
    // Should add: public init(name: String) { self.name = name }
}
```

## Architecture Patterns

### Preferred: MVVM with Combine/SwiftUI
- Use `@StateObject` for view models in SwiftUI
- Use `@Published` for observable properties
- Keep views thin, move logic to ViewModels

### UIKit Projects
- Use MVVM or VIP (Clean Architecture)
- Avoid massive view controllers
- Separate concerns: View, ViewModel/Presenter, Service

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

## Git Workflow

### Branch Naming
- Feature: `feature/description` or `feature/developer-name/ticket-number`
- Bug fix: `fix/description`
- Hotfix: `hotfix/description`

### Commit Messages
- Use conventional commits: `feat:`, `fix:`, `refactor:`, `test:`, `docs:`
- Be descriptive: `feat: add user authentication flow`
- Reference tickets when applicable: `fix: resolve crash on launch (#123)`

## Communication Guidelines with Claude

### Message Prefixes

Use these prefixes at the start of your messages to quickly indicate your intent:

#### Scope Limiting

You can limit the scope of changes by adding scope specifiers after the prefix tag:

**Syntax:**
```
[prefix]<dir:path file:FileA:10-20 class:ClassA func:functionName> Your request description
```

**Parameters:**
- `dir:` - Limit changes to a specific directory path
- `file:` - Limit changes to specific files (comma-separated, no spaces)
  - With line numbers: `file:Name.swift:10-20` (lines 10 to 20)
  - From line onwards: `file:Name.swift:50` (from line 50)
- `class:` - Limit changes to specific classes (comma-separated, no spaces)
- `func:` - Limit changes to specific functions/methods (comma-separated, no spaces)

**Examples:**
- `[fix]<file:BikeConnectionManager.swift> Memory leak in connection handling`
- `[imp]<class:UserViewModel,ProfileViewModel> Better error handling`
- `[add]<dir:Features/Authentication> Biometric authentication support`
- `[rm]<file:LegacyAPI.swift,OldNetworkManager.swift> Remove deprecated networking code`
- `[fix]<file:NetworkManager.swift:45-80> Fix error handling in this section`
- `[imp]<func:connectDevice,disconnectDevice> Improve connection logic`
- `[review]<file:UserService.swift:100-150 func:fetchUser> Check this specific area`

**Claude's response:**
- Only search and modify within the specified scope
- Ignore similar code outside the scope
- If the scope is too restrictive, mention what was skipped

#### Read-Only Mode (Dry Run)

**ANY command can be modified with `:read` suffix to prevent code changes.**

**Syntax:**
```
[prefix:read] Your request description
[prefix:read]<scope> Your request description
```

**Behavior:**
When `:read` is added to any command, Claude will:
- ✅ Analyze the code
- ✅ Explain what changes would be made
- ✅ Provide detailed plan of action
- ✅ Show code examples of proposed changes
- ❌ **NEVER modify any files**
- ❌ **NEVER write, edit, or delete code**

**Examples:**
- `[fix:read] Memory leak in BikeConnectionManager` - Explain how to fix, don't fix it
- `[imp:read]<func:connectDevice> Better error handling` - Describe improvements, don't implement
- `[add:read] Biometric authentication` - Design the feature, don't code it
- `[file-rm:read]<LegacyAPI.swift> Remove this file` - Show what would be affected, don't delete
- `[review:read]<file:Manager.swift>` - Same as `[review]` (already read-only)
- `[search:read] Find all Bluetooth code` - Same as `[search]` (already read-only)

**Note:** Commands like `[review]`, `[search]`, and `[list-cmd]` are already read-only by nature, but `:read` can still be added for consistency.

---

#### `[fix]` - Bug Fix Request
Indicates that there is a bug or error in the code that needs to be fixed.

**Examples:**
- `[fix] The app crashes when tapping the login button`
- `[fix] Memory leak in BikeConnectionManager`
- `[fix] UI not updating after data refresh`

**Claude's response:**
- Focus on identifying the root cause
- Prioritize correctness and stability
- Add defensive checks if needed
- Consider edge cases that might cause similar issues

#### `[imp]` - Code Improvement Request
Indicates that the code works but could be better (performance, readability, architecture, etc.).

**Examples:**
- `[imp] This method is too long and complex`
- `[imp] Can we make this more memory efficient?`
- `[imp] Better error handling for network requests`

**Claude's response:**
- Focus on code quality and best practices
- Suggest modern Swift patterns
- Consider performance optimizations
- Improve maintainability and readability
- Ensure access control is appropriate

#### `[add]` - Feature Implementation Request
Indicates that you want to add new code to implement a new feature or requirement.

**Examples:**
- `[add] Add biometric authentication support`
- `[add] Implement offline data caching`
- `[add] Add dark mode support to settings screen`

**Claude's response:**
- Design the feature architecture first
- Follow established patterns in the codebase
- Consider scalability and future extensions
- Implement with proper error handling
- Add appropriate tests
- Ensure proper access control from the start

#### `[rm]` - Feature Removal Request
Indicates that you want to remove code to eliminate a feature or requirement.

**Examples:**
- `[rm] Remove deprecated authentication method`
- `[rm] Delete unused analytics tracking code`
- `[rm] Remove legacy Bluetooth connection logic`

**Claude's response:**
- Identify all related code (views, models, services, tests)
- Check for dependencies and usages
- Remove carefully to avoid breaking other features
- Clean up related resources (assets, localization strings)
- Update documentation if needed
- Suggest running tests after removal

#### `[review]` - Code Review Request
Indicates that you want a comprehensive code review without making changes.

**Examples:**
- `[review]<file:BikeConnectionManager.swift> Check for memory leaks and thread safety`
- `[review]<class:NetworkClient> Review error handling and performance`
- `[review]<func:processData> Is this implementation efficient?`

**Claude's response:**
- Analyze code for potential issues
- Check for memory leaks (retain cycles)
- Review thread safety and concurrency
- Verify error handling
- Check for force unwraps and unsafe code
- Suggest improvements but don't implement them
- Provide specific line references for issues found

#### `[search]` - Search Codebase
Indicates that you want to search for code patterns, usages, or implementations.

**Examples:**
- `[search] All usages of deprecated API`
- `[search] Where is Bluetooth connection handled?`
- `[search] Find all ViewModels that use Combine`

**Claude's response:**
- Search the codebase for matching patterns
- List all relevant files and locations
- Provide context for each match
- Summarize findings
- Do not make any modifications

#### `[add-note]` - Add Comments/Documentation
Indicates that you want to add code comments or documentation.

**Examples:**
- `[add-note]<class:BikeConnectionManager> Add class documentation`
- `[add-note]<func:connectDevice> Explain the connection flow`
- `[add-note]<file:NetworkManager.swift:45-80> Document this complex logic`

**Claude's response:**
- Add clear, concise comments
- Explain WHY, not just WHAT
- Follow documentation comment standards (///)
- Add parameter descriptions for public APIs
- Include usage examples where helpful

#### `[file-add]<BFile>` - Add New File
Indicates that you want to create a new file.

**Examples:**
- `[file-add]<BiometricAuthService.swift> Create service for biometric authentication`
- `[file-add]<UserProfileViewModel.swift> Add view model for user profile screen`
- `[file-add]<NetworkConfig.swift> Add network configuration file`

**Claude's response:**
- Create the file with appropriate boilerplate
- Follow naming conventions and project structure
- Add proper imports and access control
- Include basic documentation
- Suggest where to register/use the new file if needed

#### `[file-rm]<AFile>` - Delete File
Indicates that you want to delete an existing file.

**Examples:**
- `[file-rm]<LegacyAuthManager.swift> Remove deprecated authentication manager`
- `[file-rm]<OldNetworkLayer.swift> Delete old networking code`
- `[file-rm]<UnusedViewModel.swift> Remove unused view model`

**Claude's response:**
- Check for usages and dependencies before deletion
- Warn if the file is still being used elsewhere
- Remove related imports from other files
- Suggest cleanup of related resources
- Confirm deletion safety

#### `[file-rn]<AFile,BFile>` - Rename File
Indicates that you want to rename a file from AFile to BFile.

**Examples:**
- `[file-rn]<UserManager.swift,UserService.swift> Rename to better reflect its purpose`
- `[file-rn]<Old_ViewController.swift,ViewController.swift> Remove Old_ prefix`
- `[file-rn]<BikeManager.swift,DeviceManager.swift> Rename for broader scope`

**Claude's response:**
- Check if file exists before renaming
- Update all imports and references in other files
- Ensure class/struct names match new filename if needed
- Update related documentation or comments
- Verify no broken references after rename

#### `[file-mv]<AFile,BFile>` - Move/Rename File (Alias)
Alias for `[file-rn]`. Works exactly the same as rename.

**Examples:**
- `[file-mv]<UserManager.swift,UserService.swift> Rename to better reflect its purpose`
- `[file-mv]<Old_ViewController.swift,ViewController.swift> Remove Old_ prefix`

**Claude's response:**
- Same as `[file-rn]`

#### `[list-cmd]` - List Available Commands
Display a summary of all custom message prefix commands.

**Examples:**
- `[list-cmd]`
- `[list-cmd] Show me what commands are available`

**Claude's response:**
- Display a concise list of all message prefixes
- Include brief descriptions for each command
- Show syntax examples for scope limiting and file operations
- Remind about the conventions document location

---

### When Requesting Code Changes

**DO:**
- Provide context: "In the `UserProfileViewController`, I need to..."
- Specify the file path if known
- Mention related classes or dependencies
- Share error messages or logs in full

**DON'T:**
- Say "fix the code" without specifying what's wrong
- Assume Claude knows the current state without reading files
- Skip providing error context

### When Reviewing Code

**Ask Claude to:**
- Check for memory leaks (retain cycles)
- Review thread safety (@MainActor usage)
- Verify error handling
- Check for force unwraps
- Look for performance issues

### Code Exploration

**Instead of:** "Where is the login code?"
**Say:** "Find all files related to user authentication and login flow"

**Instead of:** "What does this do?"
**Say:** "Explain the purpose of `BikeComputerConnectionContext.swift` and how it manages device connections"

## Common iOS Patterns to Follow

### Singleton Pattern
```swift
// Good: Thread-safe singleton
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

## Performance Considerations

- Use `lazy var` for expensive computations
- Avoid doing heavy work on main thread
- Use `@MainActor` for UI updates
- Profile with Instruments before optimizing
- Consider memory impact of closures and captures

## Security

- Never commit API keys or secrets
- Use Keychain for sensitive data
- Validate all user input
- Use HTTPS for network calls
- Enable App Transport Security (ATS)

## Localization

- All user-facing strings should be localized
- Use `NSLocalizedString` or SwiftGen
- Support right-to-left languages when needed

## Accessibility

- Add accessibility labels to UI elements
- Support Dynamic Type
- Test with VoiceOver
- Ensure sufficient color contrast

## Documentation

### Code Comments
```swift
// Good: Explain WHY, not WHAT
// Do not using a delayed dispatch to prevent animation conflicts unless nessasary
// use 
self.updateUI()
// instead of
DispatchQueue.main.asyncAfter(deadline: .now() + 0.1) {
    self.updateUI()
}
// unless nessasary

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

## Xcode Settings

- Enable strict concurrency checking
- Use SwiftLint for consistent style
- Enable "Treat Warnings as Errors" for release builds
- Use xcconfig files for build configurations

---

**Version:** 1.0
**Last Updated:** 2026-01-19
**Maintainer:** Your Name
