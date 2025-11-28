# Swift 6.3 Success Criteria Checklist

**Document Purpose**: Reference standard for code quality evaluation and compliance scoring  
**Target**: Swift 6.3 best practices and modern iOS/macOS development

---

## DOCUMENT STRUCTURE

This checklist defines **16 quality criteria** used by all refactoring tools to evaluate Swift code. Each criterion includes:
- **Evaluation rules**: Specific items to check
- **Pass conditions**: When code meets the criterion
- **Fail conditions**: When code violates the criterion
- **Examples**: Good vs bad patterns
- **References**: Official documentation links

**Usage**: Tools reference criteria by ID (#1-#16) for compliance tracking and reporting.

---

## CRITERION #1: Strict Concurrency Compliance

**Category**: Swift 6.3 Concurrency Safety  
**Priority**: CRITICAL (compilation requirement)

### Evaluation Rules

```
CHECK shared mutable state:
  FOR EACH static var OR class-level var:
    VERIFY protected by:
      - actor isolation
      - @MainActor annotation
      - @TaskLocal wrapper
      - OSAllocatedUnfairLock
    
    IF no protection:
      FAIL: "Unprotected global mutable state"

CHECK Sendable conformance:
  FOR EACH type crossing concurrency boundary:
    // Boundaries: Task {}, actor methods, async functions
    
    VERIFY type conforms to Sendable:
      IF struct OR enum:
        CHECK all properties are Sendable
      ELSE IF class:
        REQUIRE explicit Sendable conformance
        OR @unchecked Sendable with inline justification
    
    IF not Sendable:
      FAIL: "Non-Sendable type crossing boundary"

CHECK data race warnings:
  COMPILE with: swift build -Xswiftc -strict-concurrency=complete
  
  IF warnings detected:
    FAIL: "Data race violations present"
  ELSE:
    PASS: "Strict concurrency clean"

CHECK @MainActor coverage:
  FOR EACH UI-touching code:
    // ObservableObject, UIViewController, UIView, SwiftUI View body
    
    VERIFY @MainActor annotation present
    
    IF missing:
      FAIL: "Missing @MainActor on UI code"

CHECK @unchecked Sendable justification:
  FOR EACH @unchecked Sendable:
    VERIFY inline comment within 1 line explaining:
      WHY automatic checking bypassed
      WHAT makes this type safe despite @unchecked
    
    IF no comment:
      FAIL: "Unjustified @unchecked Sendable"

CHECK closure Sendable semantics:
  FOR EACH closure crossing boundary:
    VERIFY marked @Sendable OR
    VERIFY captures reviewed for thread-safety
    
    IF uncertain:
      FLAG: "Review closure capture semantics"
```

### Pass Conditions
- ✅ Zero data race warnings in strict concurrency mode
- ✅ All types crossing boundaries are Sendable
- ✅ All UI code has @MainActor
- ✅ Global state protected by actors or proper synchronization
- ✅ @unchecked Sendable has inline justification

### Fail Conditions
- ❌ Data race warnings from compiler
- ❌ Non-Sendable types in async contexts
- ❌ UI code missing @MainActor
- ❌ Unprotected global mutable state
- ❌ @unchecked Sendable without comment

### Code Examples

```swift
// ✅ GOOD: Sendable struct
struct User: Codable, Sendable {
    let id: String
    let name: String
    // All properties are Sendable
}

// ❌ BAD: Non-Sendable type in Task
class UserCache { // Not Sendable
    var users: [User] = []
}

Task {
    await process(cache) // ❌ Compile error in Swift 6
}

// ✅ GOOD: @MainActor on ViewModel
@MainActor
class UserViewModel: ObservableObject {
    @Published var user: User?
}

// ❌ BAD: Missing @MainActor
class UserViewModel: ObservableObject { // ❌ No @MainActor
    @Published var user: User?
}

// ✅ GOOD: Justified @unchecked Sendable
final class Configuration: @unchecked Sendable {
    // Safe: Immutable after init, never mutated
    private let settings: [String: Any]
    
    init(settings: [String: Any]) {
        self.settings = settings
    }
}

// ❌ BAD: Unjustified @unchecked
final class Cache: @unchecked Sendable { // ❌ No comment
    var data: [String: Data] = [:] // Mutable!
}
```

**References**: [Swift 6 Release Notes](https://swift.org/blog/swift-6-released/), WWDC 2024 "Swift Concurrency"

---

## CRITERION #2: Modern Async/Await Patterns

**Category**: Concurrency Modernization  
**Priority**: HIGH (best practice)

### Evaluation Rules

```
CHECK async/await usage:
  FOR EACH asynchronous operation:
    IF uses completion handler:
      FLAG: "Legacy pattern - should use async/await"
    ELSE IF uses DispatchQueue.async:
      FLAG: "Legacy GCD - should use Task or actor"

CHECK Task cancellation:
  FOR EACH long-running async operation:
    VERIFY contains:
      Task.checkCancellation() OR
      Task.isCancelled checks
    
    IF missing AND operation >1 second:
      FLAG: "Missing cancellation support"

CHECK structured concurrency:
  FOR fixed parallel tasks:
    RECOMMEND: async let
    
    EXAMPLE:
      async let user = fetchUser()
      async let settings = fetchSettings()
      return try await (user, settings)
  
  FOR dynamic parallel tasks:
    RECOMMEND: withTaskGroup
    
    EXAMPLE:
      await withTaskGroup(of: Result.self) { group in
        for id in ids {
          group.addTask { await process(id) }
        }
      }

CHECK unstructured Task usage:
  FOR EACH Task.detached:
    VERIFY comment explaining why unstructured
    
    IF no comment:
      FLAG: "Unjustified Task.detached"

CHECK AsyncSequence usage:
  FOR streaming data:
    RECOMMEND: AsyncStream, AsyncThrowingStream
    VERIFY: for await loops handle cancellation
```

### Pass Conditions
- ✅ Async operations use async/await (not completion handlers)
- ✅ Long-running tasks support cancellation
- ✅ Structured concurrency (async let, withTaskGroup) used appropriately
- ✅ Task.detached justified with comments
- ✅ AsyncSequence used for streaming data

### Fail Conditions
- ❌ Completion handlers instead of async/await
- ❌ Missing cancellation in long operations
- ❌ Unjustified Task.detached
- ❌ Manual threading instead of structured concurrency

### Code Examples

```swift
// ❌ BAD: Completion handler
func fetchUser(completion: @escaping (User?) -> Void) {
    DispatchQueue.global().async {
        let user = // ... fetch
        DispatchQueue.main.async {
            completion(user)
        }
    }
}

// ✅ GOOD: Async/await
func fetchUser() async throws -> User {
    try await networkAPI.getUser()
}

// ✅ GOOD: Structured concurrency
async let user = fetchUser()
async let settings = fetchSettings()
let (userData, settingsData) = try await (user, settings)

// ✅ GOOD: Task cancellation
func processLargeDataset() async throws {
    for item in dataset {
        try Task.checkCancellation() // Check cancellation
        await process(item)
    }
}
```

**References**: [SE-0296 Async/Await](https://github.com/apple/swift-evolution/blob/main/proposals/0296-async-await.md), [SE-0304 Structured Concurrency](https://github.com/apple/swift-evolution/blob/main/proposals/0304-structured-concurrency.md)

---

## CRITERION #3: Safe Optional Handling

**Category**: Safety & Crash Prevention  
**Priority**: HIGH (crash prevention)

### Evaluation Rules

```
SCAN for force unwraps:
  PATTERNS: !, try!, as!
  
  FOR EACH force unwrap:
    CATEGORIZE:
      IF @IBOutlet:
        ACCEPTABLE: "IBOutlet pattern"
      ELSE IF network/user/file data:
        FAIL: "Dangerous force unwrap on untrusted data"
      ELSE:
        VERIFY inline comment within 1 line
        
        IF no comment:
          FAIL: "Unjustified force unwrap"

CALCULATE reduction:
  force_unwrap_reduction = (baseline - current) / baseline × 100
  
  TARGET: 80%+ reduction
  
  IF reduction >= 80:
    PASS: "Force unwraps minimized"
  ELSE:
    FAIL: "Insufficient force unwrap reduction"

CHECK optional handling patterns:
  RECOMMEND guard let for:
    - Early exits
    - Precondition validation
  
  RECOMMEND if let for:
    - Conditional scope-limited unwrapping
  
  RECOMMEND optional chaining (?.) for:
    - Safe property/method access
  
  RECOMMEND nil coalescing (??) for:
    - Providing defaults
```

### Pass Conditions
- ✅ Force unwraps reduced by 80%+ from baseline
- ✅ Remaining force unwraps justified with inline comments or IBOutlets
- ✅ No force unwraps on network/user/file data
- ✅ Appropriate use of guard let, if let, ?., ??

### Fail Conditions
- ❌ Force unwraps on untrusted data sources
- ❌ Unjustified force unwraps without comments
- ❌ <80% reduction in force unwraps
- ❌ try! or as! without justification

### Code Examples

```swift
// ❌ BAD: Force unwrap on network data
let user = try! decoder.decode(User.self, from: data) // CRASH RISK

// ✅ GOOD: Safe error handling
do {
    let user = try decoder.decode(User.self, from: data)
    return user
} catch {
    throw NetworkError.decodingFailed(error)
}

// ✅ GOOD: Justified force unwrap (IBOutlet)
@IBOutlet var profileLabel: UILabel! // Safe: set by storyboard

// ✅ GOOD: Guard let for early exit
guard let user = fetchedUser else {
    logger.error("User not found")
    return
}
processUser(user)

// ✅ GOOD: Optional chaining
let email = user?.profile?.email ?? "No email"
```

**References**: [Apple Swift Book - Optionals](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/#Optionals)

---

## CRITERION #4: Idiomatic Naming Conventions

**Category**: Code Clarity  
**Priority**: MEDIUM (readability)

### Evaluation Rules

```
CHECK naming patterns:
  FOR variables, constants, functions, methods:
    VERIFY: camelCase
    EXAMPLES: userProfile, fetchData(), isEnabled
  
  FOR types (class, struct, enum, protocol):
    VERIFY: PascalCase
    EXAMPLES: UserManager, NetworkError, Codable
  
  FOR boolean properties:
    VERIFY: reads as assertion
    PATTERNS: is*, has*, can*, should*
    EXAMPLES: isEnabled, hasPermission, canEdit
  
  FOR capability protocols:
    VERIFY: uses suffix -able, -ible, -ing
    EXAMPLES: Codable, Hashable, Equating

CHECK name clarity:
  AVOID abbreviations except:
    - URL, HTTP, JSON, ID, UUID, API
  
  PREFER descriptive names:
    ✅ userAuthenticationToken
    ❌ uat
  
  AVOID single-letter names except:
    - Loop indices (i, j, k)
    - Generic type parameters (T, E, V)
```

### Pass Conditions
- ✅ All names follow case conventions
- ✅ Boolean properties read as assertions
- ✅ Names are descriptive and clear
- ✅ Limited abbreviations (only standard ones)

### Fail Conditions
- ❌ Inconsistent casing (wrong case for category)
- ❌ Obscure abbreviations
- ❌ Boolean properties with unclear names
- ❌ Single-letter variables outside acceptable contexts

### Code Examples

```swift
// ✅ GOOD: Proper naming
class UserManager {
    var isAuthenticated: Bool
    var currentUser: User?
    
    func fetchUserProfile() async throws -> UserProfile {
        // ...
    }
}

// ❌ BAD: Poor naming
class usrmgr { // Wrong case
    var auth: Bool // Unclear
    var usr: User? // Abbreviation
    
    func get() async throws -> UP { // Vague, abbreviated
        // ...
    }
}

// ✅ GOOD: Boolean assertions
var hasPermission = true
var canEdit = false
var isLoading = false

// ❌ BAD: Boolean names
var permission = true // Not assertive
var edit = false // Not boolean-like
var loading = false // Not assertive
```

**References**: [Swift API Design Guidelines](https://swift.org/documentation/api-design-guidelines/)

---

## CRITERION #5: Value Types by Default

**Category**: Architecture  
**Priority**: MEDIUM (best practice)

### Evaluation Rules

```
CHECK type declaration:
  FOR EACH new data model:
    PREFER: struct
    UNLESS:
      - Objective-C interop required (NSObject subclass)
      - Inheritance needed
      - Reference semantics essential (identity matters)
  
  IF class used:
    VERIFY justification comment explaining why

CHECK enum usage:
  FOR finite sets of values:
    RECOMMEND: enum with associated values
  
  EXAMPLE:
    enum NetworkResult {
      case success(User)
      case failure(Error)
    }
```

### Pass Conditions
- ✅ Data models use struct by default
- ✅ Enums used for finite value sets
- ✅ Classes justified when used
- ✅ Copy-on-write behavior leveraged

### Fail Conditions
- ❌ Class used for simple data models without justification
- ❌ Missing enum opportunity for state machines
- ❌ Overuse of reference semantics

### Code Examples

```swift
// ✅ GOOD: Struct for data model
struct User: Codable, Sendable {
    let id: String
    let name: String
    let email: String
}

// ❌ BAD: Unnecessary class
class User { // No inheritance, no identity needs
    let id: String
    let name: String
    
    init(id: String, name: String) {
        self.id = id
        self.name = name
    }
}

// ✅ GOOD: Enum for state machine
enum LoadingState {
    case idle
    case loading
    case loaded(User)
    case failed(Error)
}

// ✅ GOOD: Justified class use
// Need reference semantics for UIKit delegate pattern
class UserViewController: UIViewController {
    weak var delegate: UserDelegate?
}
```

**References**: [Choosing Between Structures and Classes](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes)

---

## CRITERION #6: Functions Are Focused and Readable

**Category**: Code Quality  
**Priority**: MEDIUM (maintainability)

### Evaluation Rules

```
MEASURE function metrics:
  FOR EACH function/method:
    COUNT lines_of_code (exclude comments/blank)
    CALCULATE cyclomatic_complexity
    
    cyclomatic_complexity = 1 + COUNT(
      if, guard, for, while, case, &&, ||, ??
    )

EVALUATE complexity:
  IF cyclomatic_complexity > 15:
    FAIL: "Exceeds hard limit"
  ELSE IF cyclomatic_complexity > 10:
    WARN: "Above target - consider refactoring"
  ELSE:
    PASS: "Complexity acceptable"

EVALUATE length:
  IF lines_of_code > 80:
    FAIL: "Exceeds hard limit"
  ELSE IF lines_of_code > 50:
    WARN: "Above target"
    CHECK cohesiveness:
      IF function is cohesive AND clear:
        ACCEPT: "Acceptable despite length"
      ELSE:
        RECOMMEND: "Extract helper functions"
  ELSE:
    PASS: "Length acceptable"

CHECK single responsibility:
  VERIFY function does ONE thing
  
  ANTI-PATTERNS:
    - Function name contains "And"
    - Multiple unrelated operations
    - Side effects + return value
```

### Pass Conditions
- ✅ Cyclomatic complexity <10 (average)
- ✅ Function length <50 lines (average)
- ✅ Each function has single responsibility
- ✅ Logical sections separated with blank lines

### Fail Conditions
- ❌ Complexity >15 (hard limit)
- ❌ Length >80 lines (hard limit)
- ❌ Multiple responsibilities in one function
- ❌ Excessive nesting (>3 levels)

### Code Examples

```swift
// ✅ GOOD: Focused function
func validateUser(_ user: User) throws {
    guard !user.name.isEmpty else {
        throw ValidationError.emptyName
    }
    guard user.email.contains("@") else {
        throw ValidationError.invalidEmail
    }
}

// ❌ BAD: Multiple responsibilities
func processUserAndSaveAndNotify(_ user: User) { // Too much
    validate(user) // Responsibility 1
    database.save(user) // Responsibility 2
    notificationCenter.post(.userUpdated) // Responsibility 3
}

// ✅ GOOD: Decomposed
func processUser(_ user: User) throws {
    try validateUser(user)
    saveUser(user)
    notifyUserUpdated()
}
```

**References**: Clean Code principles, Apple sample code

---

## CRITERION #7: Composable Error Handling

**Category**: Error Management  
**Priority**: HIGH (robustness)

### Evaluation Rules

```
CHECK error types:
  FOR EACH custom error:
    VERIFY conforms to Error protocol
    VERIFY has descriptive cases
  
  EXAMPLE:
    enum NetworkError: Error {
      case noConnection
      case timeout
      case decodingFailed(Error)
    }

CHECK async throws:
  FOR async operations that can fail:
    PREFER: async throws
    OVER: async returning Optional or Result

CHECK error silencing:
  SCAN for: try!, try?
  
  FOR EACH occurrence:
    VERIFY inline comment justifying:
      - WHY error can be ignored
      - WHAT happens if error occurs
    
    IF no comment:
      FAIL: "Unjustified error silencing"

CHECK Result usage:
  FOR non-async composable errors:
    RECOMMEND: Result<Success, Failure>
```

### Pass Conditions
- ✅ Custom Error types with descriptive cases
- ✅ async throws for async operations
- ✅ try! and try? justified with comments
- ✅ Result<T, E> used appropriately

### Fail Conditions
- ❌ Generic Error without context
- ❌ try! or try? without justification
- ❌ Swallowed errors without handling
- ❌ Optional return instead of throws

### Code Examples

```swift
// ✅ GOOD: Custom error type
enum NetworkError: Error {
    case noConnection
    case timeout(duration: TimeInterval)
    case decodingFailed(underlying: Error)
}

// ✅ GOOD: Async throws
func fetchUser() async throws -> User {
    try await networkAPI.getUser()
}

// ❌ BAD: Error silencing
let user = try! decoder.decode(User.self, from: data) // NO COMMENT

// ✅ GOOD: Justified try!
let user = try! decoder.decode(User.self, from: data)
// Safe: Data from local JSON file, validated at compile time

// ✅ GOOD: Result for non-async
func parseUser(data: Data) -> Result<User, NetworkError> {
    do {
        let user = try decoder.decode(User.self, from: data)
        return .success(user)
    } catch {
        return .failure(.decodingFailed(underlying: error))
    }
}
```

**References**: [SE-0235 Result Type](https://github.com/apple/swift-evolution/blob/main/proposals/0235-add-result.md)

---

## CRITERION #8: Memory Management & Retain Cycles

**Category**: Memory Safety  
**Priority**: HIGH (leak prevention)

### Evaluation Rules

```
CHECK closure captures:
  FOR EACH escaping closure:
    IF captures self:
      VERIFY: [weak self] OR [unowned self]
    
    IF [unowned self]:
      VERIFY comment explaining why lifetime guaranteed
    
    IF no weak/unowned:
      FLAG: "Potential retain cycle"

CHECK delegate pattern:
  FOR EACH delegate property:
    VERIFY: weak modifier
  
  EXAMPLE:
    weak var delegate: MyDelegate?

CHECK Combine:
  FOR EACH publisher subscription:
    VERIFY: .store(in: &cancellables)
  
  IF using AnyCancellable:
    VERIFY: stored or cancelled
```

### Pass Conditions
- ✅ Escaping closures use [weak self]
- ✅ Delegates declared weak
- ✅ Combine subscriptions managed
- ✅ No strong reference cycles

### Fail Conditions
- ❌ self captured strongly in escaping closures
- ❌ Delegates not weak
- ❌ Unmanaged subscriptions
- ❌ Reference cycles in object graphs

### Code Examples

```swift
// ✅ GOOD: Weak self
func observeUpdates() {
    service.onUpdate { [weak self] data in
        self?.handleUpdate(data)
    }
}

// ❌ BAD: Strong capture
func observeUpdates() {
    service.onUpdate { data in
        self.handleUpdate(data) // Retain cycle!
    }
}

// ✅ GOOD: Weak delegate
protocol UserDelegate: AnyObject {
    func userDidUpdate()
}

class UserManager {
    weak var delegate: UserDelegate?
}

// ❌ BAD: Strong delegate
class UserManager {
    var delegate: UserDelegate? // Should be weak
}
```

**References**: [Apple ARC Documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting/)

---

## CRITERION #9: Dependency Injection for Testability

**Category**: Testing & Architecture  
**Priority**: MEDIUM (testability)

### Evaluation Rules

```
CHECK dependencies:
  FOR EACH type with dependencies:
    VERIFY dependencies injected via init
    VERIFY dependencies are protocols
    
    IF singleton used:
      FLAG: "Hidden dependency - use injection"
    
    IF global state used:
      FLAG: "Untestable global dependency"

CHECK protocol abstraction:
  FOR EACH injected dependency:
    VERIFY: protocol type (not concrete)
  
  ENABLES: Mock/stub implementations
```

### Pass Conditions
- ✅ Dependencies injected via initializer
- ✅ Dependencies defined as protocols
- ✅ No hidden singletons or global state
- ✅ Mock/stub implementations possible

### Fail Conditions
- ❌ Singletons instead of injection
- ❌ Concrete types instead of protocols
- ❌ Hidden dependencies
- ❌ Untestable hard-coded dependencies

### Code Examples

```swift
// ✅ GOOD: Protocol-based injection
protocol NetworkService {
    func fetchUser() async throws -> User
}

class UserViewModel {
    private let networkService: NetworkService
    
    init(networkService: NetworkService) {
        self.networkService = networkService
    }
}

// ❌ BAD: Singleton dependency
class UserViewModel {
    func fetchUser() {
        NetworkManager.shared.getUser() // Hidden dependency
    }
}

// ✅ GOOD: Testable with mock
class MockNetworkService: NetworkService {
    func fetchUser() async throws -> User {
        User(id: "test", name: "Test User")
    }
}

let viewModel = UserViewModel(networkService: MockNetworkService())
```

**References**: WWDC Testing sessions, [Swift Testing Documentation](https://developer.apple.com/documentation/testing)

---

## CRITERION #10: Modern Testing Practices

**Category**: Testing  
**Priority**: MEDIUM (quality assurance)

### Evaluation Rules

```
FOR new test code:
  RECOMMEND: Swift Testing framework
  
  PATTERNS:
    - @Test attribute (not XCTestCase methods)
    - @Suite for grouping
    - #expect and #require (not XCTAssert*)
    - Parameterized tests
    - Native async/await

FOR existing XCTest:
  ACCEPTABLE: Continue using XCTest
  NOTE: Migration optional, not required
  
FOR third-party frameworks:
  ACCEPTABLE: Quick, Nimble, Snapshot Testing
  IF team already uses them
```

### Pass Conditions
- ✅ New tests use modern framework (Swift Testing preferred)
- ✅ Existing tests maintained consistently
- ✅ CI/CD supports chosen framework
- ✅ Tests are maintainable and clear

### Fail Conditions
- ❌ Mixing frameworks without rationale
- ❌ Deprecated testing patterns
- ❌ No CI/CD support for framework

### Code Examples

```swift
// ✅ GOOD: Swift Testing (iOS 17+)
import Testing

@Suite("User Tests")
struct UserTests {
    @Test("User initialization")
    func testUserInit() {
        let user = User(id: "123", name: "Test")
        #expect(user.id == "123")
        #expect(user.name == "Test")
    }
    
    @Test("Async user fetch")
    func testFetchUser() async throws {
        let user = try await fetchUser(id: "123")
        #require(user != nil)
    }
}

// ✅ ACCEPTABLE: XCTest (existing codebase)
import XCTest

class UserTests: XCTestCase {
    func testUserInit() {
        let user = User(id: "123", name: "Test")
        XCTAssertEqual(user.id, "123")
    }
}
```

**References**: [Swift Testing Documentation](https://developer.apple.com/documentation/testing), WWDC 2024 "Meet Swift Testing"

---

## CRITERIA #11-16 (Abbreviated Format)

Due to length constraints, the remaining criteria follow the same structure. See full document for complete details.

### #11: Type Safety with `some` and `any`
- Use generics over Any/AnyObject
- `some Protocol` for opaque return types
- `any Protocol` for heterogeneous collections

### #12: Swift Macro Usage
- @Observable (iOS 17+) over ObservableObject
- @Model for SwiftData
- #Preview for SwiftUI
- Review macro-generated code

### #13: Codable for Serialization
- Conform to Codable for JSON/Plist
- CodingKeys for custom mappings
- Proper date/key decoding strategies
- Off-thread decoding for large payloads

### #14: Security Best Practices
- Zero hardcoded secrets
- Keychain for sensitive data
- Input validation
- HTTPS/ATS enforcement

### #15: SwiftUI View Best Practices
- Small,composable views
- @Observable (iOS 17+) over ObservableObject
- @State for local state
- Pure view body computation

### #16: Documentation and Comments
- DocC comments on public APIs
- MARK sections for organization
- Explain "why" not "what"
- No commented-out code

---

## COMPLIANCE CALCULATION

```
FOR EACH file being evaluated:
  
  IDENTIFY applicable criteria:
    FOR criterion IN [1..16]:
      IF criterion applies to file:
        ADD to applicable_list
      ELSE:
        MARK as N/A (exclude from scoring)
  
  EVALUATE each applicable criterion:
    FOR criterion IN applicable_list:
      COUNT total_items_in_criterion
      COUNT passed_items
      
      criterion_pass_rate = passed_items / total_items × 100
      
      IF criterion_pass_rate >= 85:
        STATUS = ✅ PASSED
      ELSE IF criterion_pass_rate >= 70:
        STATUS = ⚠️ WARNING
      ELSE:
        STATUS = ❌ FAILED
  
  CALCULATE overall_compliance:
    total_passed = SUM(all passed_items across criteria)
    total_applicable = SUM(all total_items across criteria)
    
    overall_percentage = (total_passed / total_applicable) × 100
  
  DETERMINE compliance status:
    IF overall_percentage >= 85:
      RESULT = ✅ COMPLIANT (meets target)
    ELSE IF overall_percentage >= 70:
      RESULT = ⚠️ PARTIAL (needs improvement)
    ELSE:
      RESULT = ❌ NON-COMPLIANT (significant issues)
```

**Target**: 85%+ overall compliance per file

---

## FORBIDDEN ACTIONS (NEVER DO)

```
❌ NEVER add new features or functionality
❌ NEVER change public APIs without explicit permission
❌ NEVER alter business logic or behavior
❌ NEVER remove code without understanding purpose
❌ NEVER break existing tests
❌ NEVER modify database schemas
❌ NEVER change third-party library code
❌ NEVER add new dependencies
❌ NEVER optimize prematurely without measurement
❌ NEVER rewrite working code for style alone
❌ NEVER change file/folder structure without permission
❌ NEVER modify compliance/regulatory code
❌ NEVER auto-fix warnings blindly
❌ NEVER change localization strings
❌ NEVER proceed when code intent is unclear
```

---

## SUCCESS METRICS

### Hard Requirements (Must Pass ALL)
```
✅ Zero compiler errors or warnings
✅ Zero data race violations (Swift 6 strict concurrency)
✅ 100% existing tests passing
✅ Behavior preservation verified
```

### Quality Gates (Must Meet 4/5)
```
✅ Force unwraps reduced 80%+ (remaining justified)
✅ Public API documentation 100% coverage
✅ Checklist compliance 85%+ per file
✅ Cyclomatic complexity average <10
✅ Code coverage maintained or improved
```

### Qualitative Assessment
```
🔍 Code more readable than before
🔍 Code more testable than before
🔍 Code more maintainable than before
```

---

*This Success Criteria Checklist is the authoritative reference for all refactoring tools. Update as Swift evolves and new best practices emerge.*

*Last Updated: 2025-11-26*
