---
description: Context Intake Model for Swift Refactor Agent
---

# CONTEXT INTAKE PROTOCOL

## PURPOSE

This protocol defines mandatory context acquisition requirements for Phase 1 (Context Intake) of the Swift 6.3 refactoring workflow. Execute these directives systematically to ensure complete project understanding before code analysis.

---

## INPUT PARAMETERS

### Required Inputs
```
{target_file}       : Absolute path to the primary Swift file to be refactored.
                      (Provided via command: /context-intake @{target_file})
```

---

## 0. SESSION INITIALIZATION (Artifact Archival)

**CRITICAL STEP**: Before gathering new context, you MUST clear the workspace of previous session data to prevent hallucinations.

```
ACTION: CHECK for existing artifacts in .agent/artifacts/
  - context_summary.md
  - analysis_report.md
  - review_report.md
  - refactor_plan.md
  - refactor_execution_log.md
  - validation_report.md

IF artifacts exist:
  1. GENERATE timestamp: {YYYY-MM-DD_HH-mm-ss}
  2. CREATE directory: .agent/artifacts/history/{timestamp}/
  3. MOVE all existing .md files from .agent/artifacts/ to history folder
  4. LOG: "Previous session artifacts archived to history/{timestamp}/"

RESULT: .agent/artifacts/ directory should be clean (except for history folder & .agent/artifacts/introduction.md).
```

---

# MANDATORY ARTIFACTS

## 1. Project-Level Context (Foundation)

Acquire these artifacts to establish baseline project understanding:

### 1.1 Documentation Files


**PRODUCT_SPEC.md**
```
LOCATION: Project root or `/docs`
PURPOSE: Understand business requirements and feature specifications
IF MISSING: Note in Context Summary → proceed with technical focus only
```

**ENGINEERING_STANDARDS.md**
```
LOCATION: Project root or `/docs`
PURPOSE: Extract code style rules, patterns, technical constraints
IF MISSING: Infer standards from existing codebase patterns
```

**CODE_TEMPLATES.md**
```
LOCATION: Project root or `/docs`
PURPOSE: Identify standard implementations for common patterns
IF MISSING: Extract patterns from codebase analysis
```

### 1.2 Architecture Definitions

**Architecture Documentation**
```
SEARCH LOCATIONS: 
  - /docs/ARCHITECTURE.md
  - /docs/architecture/
  - /.github/
  - Project root

PURPOSE: Map layers, modules, component relationships
IF MISSING: Infer from code structure → document discovered architecture in Context Summary
```

**Module Organization**
```
EXAMINE:
  - Folder structure hierarchy
  - File naming conventions
  - Module maps and targets

OUTPUT: Document organization pattern (feature-based, layer-based, domain-based)
```

### 1.3 Build Configuration

**Package.swift** (Swift Package Manager)
```
EXTRACT:
  - Swift tools version
  - External package dependencies
  - Local target definitions
  - Platform requirements

CRITICAL: Identify Swift version (must be 6.3)
```

**Project.pbxproj / .xcodeproj** (Xcode)
```
PARSE:
  - Build configurations
  - Target schemes
  - Compiler flags (especially strict concurrency mode)
  - Linked frameworks

CRITICAL: Verify Swift 6.3 strict concurrency enabled
```

**.xcworkspace**
```
IF PRESENT: Note CocoaPods or multi-module setup
PURPOSE: Understand workspace-level dependencies
```

**Info.plist** (App Targets)
```
RELEVANCE: When refactoring app lifecycle or capability-related code
EXTRACT: App capabilities, permissions, configuration keys
```

### 1.4 Dependency Manifests

**Package.resolved** (SPM Lock File)
```
PURPOSE: Determine exact dependency versions
USE CASE: Assess API compatibility constraints
```

**Podfile & Podfile.lock** (CocoaPods)
```
PURPOSE: Document third-party dependencies and versions
EXTRACT: Pod specifications and version locks
```

**Framework Inventory**
```
CATALOG: Major frameworks in active use
EXAMPLES: SwiftUI, UIKit, Combine, AsyncSequence, SwiftData, CoreData, Realm
OUTPUT: List in Context Summary for dependency awareness
```

### 1.5 Quality Tooling

**.swiftlint.yml**
```
IF PRESENT: 
  - Read complete configuration
  - Ensure refactored code complies with rules
  - Document custom rules in Context Summary
```

**Compiler Settings**
```
LOCATE: Build settings or swift-settings files
CRITICAL CHECKS:
  - Swift 6.3 strict concurrency mode: ENABLED / DISABLED
  - Swift language version
  - Other compiler flags (-warnings-as-errors, etc.)
```

### 1.6 Testing Infrastructure

**Test Directory Structure**
```
SEARCH LOCATIONS:
  - Tests/
  - *Tests/ (subdirectories)
  - Test targets in Xcode project

PURPOSE: Understand testing organization and coverage patterns
```

**Testing Framework**
```
IDENTIFY:
  - XCTest (standard)
  - Swift Testing (modern)
  - Quick/Nimble (BDD)
  - Custom test utilities

OUTPUT: Document framework in Context Summary for test writing guidance
```

---

## 2. Task-Specific Context (Per Refactor Request)

For EACH refactoring task, acquire these localized artifacts:

### 2.1 Target Code Files

**Primary Target File(s)**
```
SOURCE: {target_file} input parameter
ACTIONS:
  1. Read complete file contents → use view_file
  2. Generate outline structure → use view_file_outline
  3. Parse imports, types, functions, properties
```

**Focused Symbols** (if user specified)
```
IF USER SPECIFIES: Specific functions, classes, regions
THEN: Use view_code_item for targeted analysis
```

### 2.2 Type System Context

**Protocol Definitions**
```
IF target conforms to protocols
THEN request ALL protocol definitions
  - Protocol requirements
  - Associated types
  - Default implementations (extensions)
```

**Superclass Hierarchy**
```
IF target inherits from class
THEN request:
  - Complete superclass definition
  - Superclass chain up to Foundation/UIKit base classes
  - Override requirements
```

**Associated Data Types**
```
REQUEST types used as:
  - Properties
  - Method parameters
  - Return types
  - Generic constraints

EXCLUDE: Swift Standard Library types (String, Int, Array, etc.)
```

**Type Extensions**
```
SEARCH PATTERN: `extension {TargetTypeName}`
SCOPE: Entire project codebase
PURPOSE: Capture ALL functionality, not just main definition
TOOL: grep_search
```

### 2.3 Outgoing Dependencies

**Imported Modules**
```
EXTRACT: All `import` statements from target file
CATEGORIZE:
  - Standard Library (Foundation, Swift)
  - Apple Frameworks (SwiftUI, UIKit, Combine)
  - Third-party (from dependencies)
  - Internal modules (project-specific)
```

**Injected Dependencies**
```
SEARCH IN:
  - Initializer parameters
  - Property declarations with injection

REQUEST: Definitions of ALL injected types
PURPOSE: Understand dependency contracts and lifetimes
```

**Protocol-Based Dependencies**
```
IDENTIFY: Types referenced via protocols (abstraction pattern)
REQUEST:
  - Protocol definitions
  - Concrete implementations
  - Factory or container patterns
```

### 2.4 Incoming Dependencies

**Call Sites**
```
SEARCH PATTERN: Type name, function names
TOOL: grep_search or codebase_search
SCOPE: 
  - Entire project for public/open APIs
  - Module only for internal APIs

PURPOSE: Map consumption patterns and change impact
```

**Subclass Hierarchy**
```
IF target is a class
THEN search pattern: `: {TargetClassName}`
PURPOSE: Identify all types inheriting from target
TOOL: grep_search
```

**Protocol Adopters**
```
IF target is a protocol
THEN search pattern: `: {ProtocolName}`, `, {ProtocolName}`
PURPOSE: Locate all conforming types
SCOPE: Entire codebase
```

### 2.5 Test Coverage

**Unit Test Files**
```
SEARCH PATTERNS:
  - {TargetName}Tests.swift
  - {TargetName}Spec.swift
  - Test{TargetName}.swift

LOCATIONS: Test targets, Tests/ directory
IF NOT FOUND: Mark as HIGH RISK in Context Summary
```

**Integration Test Files**
```
SEARCH: UI tests, end-to-end tests referencing target
PURPOSE: Understand cross-component interactions
```

**Test Utilities**
```
SEARCH PATTERNS:
  - Mock{TargetName}
  - {TargetName}Stub
  - {TargetName}Fake

PURPOSE: Understand test double patterns for maintaining consistency
```

### 2.6 Presentation Layer Context (ViewModels/Controllers)

**Associated View Files**
```
IF refactoring ViewModel/ViewController
THEN request corresponding View:
  - FooViewModel → FooView (SwiftUI)
  - FooViewController → FooView.swift or storyboard

PURPOSE: Understand UI bindings and presentation logic
```

**Navigation Context**
```
IF coordinator pattern detected
THEN request:
  - Coordinator files
  - Navigation manager definitions
  - Flow coordinator implementations
```

### 2.7 Data Layer Context (Models/Entities)

**Persistence Schema**
```
IF target is SwiftData model (@Model)
THEN request:
  - Related @Model classes
  - @Relationship definitions
  - ModelContainer configuration
  - Migration schemas

IF target is CoreData entity (NSManagedObject)
THEN request:
  - .xcdatamodeld schema file
  - Related entity definitions
  - Fetch request templates
```

**Serialization Logic**
```
IF target conforms to Codable
THEN check:
  - Custom CodingKeys enums
  - Custom encode/decode implementations
  - Transformable attributes

PURPOSE: Ensure refactor maintains serialization compatibility
```

**Migration Configurations**
```
IF database model
THEN search for:
  - Migration folders
  - Version policies
  - Schema evolution strategies
```

---

# DYNAMIC CONTEXT RULES

Execute these conditional logic rules during context gathering:

## Rule 1: Type Reference Resolution
```
IF target code references custom type (property, parameter, return type)
AND type NOT in Swift Standard Library
AND type NOT already in collected context
THEN:
  1. Search for type definition using grep_search or codebase_search
  2. Read type definition with view_file or view_code_item
  3. Add to context collection
  4. Apply recursive rules to newly discovered type
```

**Trigger Examples:**
- `var repository: UserRepository` → REQUEST UserRepository
- `func process(_ item: CustomItem)` → REQUEST CustomItem
- `func fetch() -> Result<User, NetworkError>` → REQUEST User AND NetworkError

---

## Rule 2: Protocol Conformance Chain
```
IF target type conforms to ≥1 protocol(s)
THEN:
  1. Request ALL protocol definitions
  2. Search for extensions providing conformance implementations
  3. Search for protocol witnesses (other types conforming to same protocols)
  
RATIONALE: Protocol changes affect ALL conforming types
```

**Example:**
```swift
class DataManager: DataProviding, Sendable { }
```
**Actions:**
1. REQUEST `DataProviding` protocol definition
2. SEARCH `grep_search(": DataProviding")` for other adopters
3. REQUEST `extension DataManager: DataProviding` if exists

---

## Rule 3: Public API Impact Analysis
```
IF target contains `public` or `open` declarations
THEN:
  1. Search ENTIRE codebase for ALL call sites (no scope limits)
  2. Use grep_search with type/method name patterns
  3. Request ALL calling files
  
IF refactor plan modifies public API
THEN:
  1. Search for ALL files importing module containing target
  2. Map complete blast radius
  3. Document breaking change impact in Risk Assessment
```

**Example:**
```swift
public protocol ViewModelProtocol { }
public func configure(with config: Config) { }
```
**Actions:**
1. SEARCH entire codebase: `grep_search("configure(with:")`
2. SEARCH protocol adopters: `grep_search(": ViewModelProtocol")`
3. MAP all module import sites

---

## Rule 4: Inheritance Hierarchy Traversal
```
IF target is class with superclass
THEN:
  1. Request complete superclass definition
  2. Request ALL overridden methods from superclass
  3. Search for ALL subclasses of target: `grep_search(": {TargetClass}")`
  
IF refactor involves changing override signatures
THEN:
  1. Request ENTIRE class hierarchy (ancestors + descendants)
  2. Map override chains
```

**Example:**
```swift
class CustomViewController: BaseViewController { }
```
**Actions:**
1. REQUEST `BaseViewController` definition
2. TRAVERSE superclass chain up to UIKit
3. SEARCH subclasses: `grep_search(": CustomViewController")`

---

## Rule 5: Concurrency Boundary Analysis
```
IF target crosses concurrency boundaries (async, actor, @MainActor)
THEN:
  1. Request ALL types interacting across boundaries
  2. CHECK Sendable conformance requirements
  3. VERIFY actor isolation contexts
  
IF target uses Sendable-constrained types
THEN:
  1. Request type definitions
  2. VERIFY explicit Sendable conformance (not assumed)
  3. CHECK for non-Sendable stored properties
```

**Example:**
```swift
actor DataCache {
    func store(_ item: CacheItem) async { }
}
```
**Actions:**
1. REQUEST `CacheItem` definition
2. VERIFY `CacheItem: Sendable` conformance exists
3. SEARCH call sites to map isolation contexts

---

## Rule 6: Dependency Injection Discovery
```
IF target has injected dependencies (init params, property injection)
THEN:
  1. Request definitions of ALL dependency types
  2. Search for dependency container or factory
  3. Search construction sites: `grep_search("{TargetType}(")`
  
PURPOSE: Understand dependency lifetime, scope, and wiring
```

**Example:**
```swift
class UserService {
    init(repository: UserRepository, 
         logger: LoggerProtocol,
         config: AppConfiguration) { }
}
```
**Actions:**
1. REQUEST `UserRepository`, `LoggerProtocol`, `AppConfiguration` definitions
2. SEARCH construction: `grep_search("UserService(")`
3. IDENTIFY dependency injection container (if exists)

---

## Rule 7: Test Coverage Mapping
```
WHEN beginning ANY refactor task
THEN:
  1. Search for test files matching patterns:
     - {TargetName}Tests.swift
     - {TargetName}Spec.swift
     - Test{TargetName}.swift
  
IF no tests found
THEN:
  1. Note in Context Summary as HIGH RISK
  2. Include test creation in Refactor Plan (Phase 4)
```

---

## Rule 8: Extension Discovery
```
IF target is type (class, struct, enum, protocol, actor)
THEN:
  1. Search ENTIRE codebase: `grep_search("extension {TargetName}")`
  2. Request ALL discovered extension files
  3. Document extension organization pattern in Context Summary
  
PURPOSE: Capture complete type functionality across file boundaries
```

---

## Rule 9: Generic Constraint Analysis
```
IF target uses generics with constraints
THEN:
  1. Request definitions of ALL constraint types
  2. Verify constraint type conformances
  3. Analyze call site usage patterns
  
EXAMPLE: `func process<T: Encodable & Sendable>(_ item: T)`
  → VERIFY Encodable, Sendable definitions
  → CHECK call sites for constraint satisfaction
```


## Rules 10-16: Additional Context Rules (Apply When Relevant)

```
Rule 10 - SwiftUI Views: Request parent/child views + state objects (@StateObject, @ObservedObject, @EnvironmentObject)

Rule 11 - Async/Await: Search await call sites, check Task cancellation patterns, identify TaskGroup usage

Rule 12 - Combine: Map publisher chains, request published value types, document subscriber sites

Rule 13 - Property Wrappers: Request custom wrapper definitions, document side effects, search usage sites

Rule 14 - Error Types: Request custom Error definitions, search catch blocks, map error propagation

Rule 15 - SwiftData: Request related @Model classes, check @Relationship macros, request ModelContainer config

Rule 16 - CoreData: Request .xcdatamodeld schema, related entities, fetch templates, migration models
```

---

# OUTPUT SPECIFICATION

## Purpose

Phase 1 produces a **Context Summary Document** that serves as the authoritative input for Phase 2 (`/analyze-context`). This document must be complete, structured, and machine-parsable.

---

## Output Format: Markdown Document

**Filename:** `context_summary.md`  
**Location:** `.agent/artifacts/` (create directory if not exists)  
**Format:** Structured Markdown with YAML frontmatter

---

## Required Document Structure

```markdown
---
phase: 1
phase_name: Context Intake
analysis_date: {ISO 8601 timestamp}
target_files: 
  - {absolute path 1}
  - {absolute path 2}
swift_version: "6.3"
strict_concurrency_enabled: {true | false | unknown}
overall_risk: {LOW | MEDIUM | HIGH}
scope: {SMALL | MEDIUM | LARGE}
---

# Context Summary: {Project Name}

**Generated:** {ISO 8601 timestamp}  
**Target Files:** {count} file(s)  
**Analysis Scope:** {SMALL | MEDIUM | LARGE}  
**Overall Risk Level:** {LOW | MEDIUM | HIGH}

---

## 1. USER REQUEST

### Refactoring Objectives
{Bullet list of user's stated goals from initial request}

### Success Criteria
{User-defined measures of success, or "Not explicitly defined"}

### Scope Boundaries
- **Files In Scope:** {list}
- **Files Out of Scope:** {list or "N/A"}
- **Known Constraints:** {list or "None specified"}

---

## 2. PROJECT FOUNDATION

### Architecture Pattern
**Detected Pattern:** {MVVM | MVC | VIPER | TCA | Custom | Unknown}  
**Confidence:** {HIGH | MEDIUM | LOW}  
**Evidence:** {Brief description of how pattern was identified}

### Swift 6.3 Adoption Status
- **Swift Version:** {version from Package.swift or project settings}
- **Strict Concurrency Mode:** {ENABLED | DISABLED | UNKNOWN}
- **Async/Await Usage:** {WIDESPREAD | PARTIAL | MINIMAL | NONE}
- **Actor Usage:** {PRESENT | ABSENT}
- **Sendable Conformance:** {EXPLICIT | INFERRED | MISSING}

### Governance Documents

| Document | Status | Location |
|----------|--------|----------|
| README.md | {FOUND \| NOT_FOUND} | {path or N/A} |
| ENGINEERING_STANDARDS.md | {FOUND \| NOT_FOUND} | {path or N/A} |
| CODE_TEMPLATES.md | {FOUND \| NOT_FOUND} | {path or N/A} |
| PRODUCT_SPEC.md | {FOUND \| NOT_FOUND} | {path or N/A} |

**Key Standards Extracted:**
{Bullet list of relevant constraints from governance docs, or "No governance documents found"}

### Project Dependencies

**Package Manager:** {SPM | CocoaPods | Both | None}

**Apple Frameworks:**
- {list of imported Apple frameworks}

**Third-Party Libraries:**
- {list with versions if available, or "None detected"}

**Internal Modules:**
- {list of project-internal modules}

### Testing Infrastructure
- **Test Framework:** {XCTest | Swift Testing | Quick/Nimble | Mixed | None}
- **Test Directory:** {path or "Not found"}
- **Test File Count:** {count}

---

## 3. TARGET FILE ANALYSIS

{FOR EACH target file, create subsection:}

### 3.{N}: `{FileName.swift}`

**Path:** `{absolute path}`  
**Type Category:** {class | struct | enum | actor | protocol | extension}  
**Access Level:** {public | open | internal | private | fileprivate}  
**Lines of Code:** {count}

#### Type Inventory
- **Primary Types:** {list of main types defined}
- **Nested Types:** {list or "None"}
- **Extensions:** {count} extension(s) found

#### Protocol Conformances
{FOR EACH type:}
- `{TypeName}`: {list of protocols, or "None"}

#### Inheritance Hierarchy
{FOR EACH class:}
- `{ClassName}` inherits from `{SuperclassName}`

#### Concurrency Characteristics
- **Async Methods:** {count}
- **Actor Isolation:** {@MainActor | actor {Name} | nonisolated | none}
- **Sendable Status:** {Explicit | Inferred | Missing | N/A}

---

## 4. DEPENDENCY MAP

### 4.1 Type Dependencies (Outgoing)

{FOR EACH custom type referenced:}

| Referenced Type | Category | Location | Status |
|----------------|----------|----------|--------|
| `{TypeName}` | {Model \| Service \| Protocol \| etc.} | `{file path}` | {RESOLVED \| MISSING} |

**Depth Level Reached:** {1 | 2 | 3}  
**Recursion Stopped:** {Yes - depth limit | No - complete}

### 4.2 Protocol Dependencies

{FOR EACH protocol conformed to:}

**Protocol:** `{ProtocolName}`
- **Definition Location:** `{file path or "Swift Standard Library"}`
- **Requirements:** {count} requirement(s)
- **Default Implementations:** {FOUND in extensions | NOT_FOUND}
- **Other Adopters:** {count} type(s) conform to this protocol

### 4.3 Incoming Dependencies (Consumers)

**Call Site Analysis:**
- **Total Usage Sites:** {count}
- **Calling Files:** {list of file paths}
- **Public API Exposure:** {YES - public/open | NO - internal/private}

{IF public API:}
**⚠️ PUBLIC API IMPACT:**
- **External Consumers:** {count} file(s) outside target
- **Breaking Change Risk:** {HIGH | MEDIUM | LOW}

### 4.4 Test Coverage

**Test Files Located:**
{list of test file paths, or "⚠️ NO TESTS FOUND - HIGH RISK"}

**Test Framework:** {XCTest | Swift Testing | etc.}  
**Test Count:** {approximate count or "Unknown"}  
**Coverage Assessment:** {HIGH | MEDIUM | LOW | UNKNOWN}

---

## 5. CONTEXT ACQUISITION TRACE

### Files Read
{Ordered list of all files read during context gathering:}
1. `{file path}` - {reason}
2. `{file path}` - {reason}
...

**Total Files Read:** {count}

### Search Operations Performed
{List of grep_search, codebase_search, find_by_name operations:}
1. **Search:** `{pattern}` → {count} results
2. **Search:** `{pattern}` → {count} results
...

### Dynamic Rules Applied
{List which of the 16 rules were triggered:}
- ✅ Rule 1: Type Reference Resolution → {count} types resolved
- ✅ Rule 2: Protocol Conformance Chain → {count} protocols analyzed
- ✅ Rule 7: Test Coverage Mapping → {TESTS_FOUND | NO_TESTS}
- ✅ Rule 8: Extension Discovery → {count} extensions found
- {etc.}

---

## 6. RISK ASSESSMENT

### Overall Risk Level: {LOW | MEDIUM | HIGH}

**Risk Calculation:**
```
Risk Factors:
- Public API changes: {YES/NO} → {+HIGH | +MEDIUM | +0}
- Missing test coverage: {YES/NO} → {+HIGH | +0}
- Complex dependencies: {YES/NO} → {+MEDIUM | +0}
- Missing governance docs: {YES/NO} → {+LOW | +0}
- Strict concurrency disabled: {YES/NO} → {+MEDIUM | +0}

TOTAL RISK: {LOW | MEDIUM | HIGH}
```

### Missing Artifacts

{IF any required context is missing:}

| Artifact | Severity | Impact |
|----------|----------|--------|
| `{TypeName}` definition | {HIGH \| MEDIUM \| LOW} | {description} |
| Test files | {HIGH \| MEDIUM \| LOW} | Cannot verify behavior preservation |
| {etc.} | | |

{IF no missing artifacts:}
✅ All required context successfully acquired.

### Assumptions Made

{IF any assumptions were necessary:}

⚠️ **ASSUMPTION:** {description of assumption}  
**Rationale:** {why assumption was made}  
**Risk if Wrong:** {impact description}

{IF no assumptions:}
✅ No assumptions required - all context verified.

### High-Risk Areas

{List areas requiring extra caution:}
1. **{Area}:** {description of risk}
2. **{Area}:** {description of risk}

---

## 7. COMPLETENESS VERIFICATION

### Phase 1 Checklist Status

```
PROJECT FOUNDATION:
  ✅ README.md located and read
  ✅ Swift version confirmed: 6.3
  ✅ Strict concurrency status: {ENABLED/DISABLED}
  ✅ Dependencies cataloged
  ✅ Architecture pattern identified

TARGET ANALYSIS:
  ✅ Target file(s) read: {count}
  ✅ Type inventory completed
  ✅ Access levels documented
  ✅ Concurrency characteristics analyzed

DEPENDENCY MAPPING:
  ✅ Type dependencies resolved (depth {1-3})
  ✅ Protocol conformances identified
  ✅ Call sites mapped
  ✅ Test files located: {YES/NO}

COMPLETENESS GATES:
  {✅ | ⚠️ | ❌} All referenced types defined
  {✅ | ⚠️ | ❌} All protocols defined
  {✅ | ⚠️ | ❌} Public API call sites mapped
  {✅ | ⚠️ | ❌} Concurrency types verified as Sendable
  {✅ | ⚠️ | ❌} Test coverage assessed
```

**Legend:**
- ✅ Complete
- ⚠️ Partial (see Missing Artifacts)
- ❌ Failed (blocking issue)

### Gaps Requiring User Input

{IF any gaps require user clarification:}

**QUESTION {N}:** {description of ambiguity}  
**Options:**
1. {option 1}
2. {option 2}

**Recommended Action:** {suggestion}

{IF no gaps:}
✅ No user input required - ready for Phase 2.

---

## 8. PHASE 2 READINESS

### Input for `/analyze-context`

This Context Summary provides Phase 2 with:
- ✅ Complete target file inventory
- ✅ Type and dependency map
- ✅ Concurrency safety baseline
- ✅ Risk assessment foundation
- ✅ Test coverage status

### Recommended Next Steps

1. **User Review:** Review this Context Summary for accuracy
2. **Gap Resolution:** Address any missing artifacts or questions above
3. **Approval Gate:** Confirm readiness to proceed
4. **Phase 2 Invocation:** Run `/analyze-context` with this summary

### Blockers (if any)

{IF blockers exist:}
❌ **BLOCKER:** {description}  
**Resolution Required:** {action needed before Phase 2}

{IF no blockers:}
✅ No blockers - ready to proceed to Phase 2.

---

**END OF CONTEXT SUMMARY**

*Generated by Phase 1: Context Intake*  
*Next Phase: `/analyze-context` for Logic & Dependency Report*
```

---

## Validation Schema

Before saving, verify the Context Summary contains:

```
MANDATORY SECTIONS (must exist):
  ✅ Section 1: USER REQUEST
  ✅ Section 2: PROJECT FOUNDATION
  ✅ Section 3: TARGET FILE ANALYSIS
  ✅ Section 4: DEPENDENCY MAP
  ✅ Section 5: CONTEXT ACQUISITION TRACE
  ✅ Section 6: RISK ASSESSMENT
  ✅ Section 7: COMPLETENESS VERIFICATION
  ✅ Section 8: PHASE 2 READINESS

MANDATORY FIELDS (must have values):
  ✅ analysis_date (YAML frontmatter)
  ✅ target_files (at least 1)
  ✅ swift_version
  ✅ overall_risk
  ✅ scope

QUALITY CHECKS:
  ✅ No placeholder text like "{TODO}" or "{FILL_IN}"
  ✅ All file paths are absolute
  ✅ All search operations documented
  ✅ Risk assessment has rationale
  ✅ Completeness checklist filled
```

---

## Output Generation Protocol

```
STEP 1: Collect all context per execution workflow
STEP 2: Populate Context Summary template with gathered data
STEP 3: Validate against schema above
STEP 4: Save to `.agent/artifacts/context_summary.md`
STEP 5: Present summary to user for review
STEP 6: WAIT for user confirmation:
        "Context Summary complete. Proceed to Phase 2? (yes/no)"
STEP 7: IF yes → Output file path for Phase 2 consumption
        IF no → Ask "What needs revision?"
```

---

## Example Output Location

```
Project Root/
├── .agent/
│   ├── artifacts/
│   │   └── context_summary.md          ← Phase 1 output (THIS FILE)
│   └── workflow/
│       ├── context-intake.md           ← Phase 1 instructions
│       └── analyze-context2.md         ← Phase 2 instructions (reads context_summary.md)
```

---

## Phase 1 → Phase 2 Handoff

**Phase 2 Invocation:**
```
User runs: /analyze-context

Phase 2 agent:
  1. READ `.agent/artifacts/context_summary.md`
  2. PARSE YAML frontmatter for metadata
  3. EXTRACT target_files list
  4. EXTRACT dependency map
  5. EXTRACT risk assessment
  6. BEGIN deep code analysis (Phase 2 logic)
  7. PRODUCE "Logic & Dependency Report"
```

**Critical:** Phase 2 MUST NOT re-gather context. It consumes Phase 1 output.

---

# EXAMPLE COMPLETE OUTPUT

Shows the structure and key sections of a Context Summary. See OUTPUT SPECIFICATION for full template.

## Example Scenario

**User Request:** "Refactor UserProfileViewModel.swift to use async/await"  
**Target File:** `/Users/project/App/ViewModels/UserProfileViewModel.swift`

---

## Generated Context Summary (Condensed)

```markdown
---
phase: 1
phase_name: Context Intake
analysis_date: 2025-11-28T06:10:00Z
target_files: 
  - /Users/project/App/ViewModels/UserProfileViewModel.swift
swift_version: "6.3"
strict_concurrency_enabled: false
overall_risk: MEDIUM
scope: SMALL
---

# Context Summary: MyApp

**Generated:** 2025-11-28T06:10:00Z  
**Target Files:** 1 file(s)  
**Overall Risk Level:** MEDIUM

## 1. USER REQUEST

### Refactoring Objectives
- Convert UserProfileViewModel to use async/await
- Add Sendable conformance for Swift 6.3
- Eliminate force unwraps

### Success Criteria
- Zero data race warnings
- All tests passing
- Force unwraps reduced by 80%+

### Scope Boundaries
- **Files In Scope:** UserProfileViewModel.swift
- **Files Out of Scope:** UserProfileView.swift
- **Known Constraints:** Maintain backward compatibility

## 2. PROJECT FOUNDATION

### Architecture Pattern
**Detected Pattern:** MVVM  
**Confidence:** HIGH  
**Evidence:** ViewModel suffix, ObservableObject conformance, @Published properties

### Swift 6.3 Adoption Status
- **Swift Version:** 6.3
- **Strict Concurrency Mode:** DISABLED (⚠️ needs enabling)
- **Async/Await Usage:** PARTIAL
- **Sendable Conformance:** MISSING

### Governance Documents
| Document | Status | Location |
|----------|--------|----------|
| README.md | FOUND | /Users/project/README.md |
| ENGINEERING_STANDARDS.md | FOUND | /Users/project/docs/ENGINEERING_STANDARDS.md |
| CODE_TEMPLATES.md | NOT_FOUND | N/A |

**Key Standards Extracted:**
- Use protocol-based dependency injection
- All ViewModels must be @MainActor
- Prefer async/await over completion handlers

## 3. TARGET FILE ANALYSIS

### 3.1: `UserProfileViewModel.swift`

**Path:** `/Users/project/App/ViewModels/UserProfileViewModel.swift`  
**Type Category:** class  
**Access Level:** internal  
**Lines of Code:** 187

#### Type Inventory
- **Primary Types:** UserProfileViewModel
- **Extensions:** 1 extension (UserProfileViewModel+Validation.swift)

#### Protocol Conformances
- `UserProfileViewModel`: ObservableObject

#### Concurrency Characteristics
- **Async Methods:** 2
- **Actor Isolation:** none (⚠️ should be @MainActor)
- **Sendable Status:** Missing

## 4. DEPENDENCY MAP

### 4.1 Type Dependencies (Outgoing)

| Referenced Type | Category | Location | Status |
|----------------|----------|----------|--------|
| `User` | Model | `/Users/project/Models/User.swift` | RESOLVED |
| `UserRepository` | Service | `/Users/project/Networking/UserRepository.swift` | RESOLVED |

**Depth Level Reached:** 2  
**Recursion Stopped:** No - complete

### 4.2 Protocol Dependencies

**Protocol:** `ObservableObject`
- **Definition Location:** Swift Standard Library (Combine)
- **Other Adopters:** 8 type(s) in project

### 4.3 Incoming Dependencies (Consumers)

**Call Site Analysis:**
- **Total Usage Sites:** 3
- **Calling Files:** UserProfileView.swift, UserProfileCoordinator.swift, Tests
- **Public API Exposure:** NO - internal

**Breaking Change Risk:** LOW

### 4.4 Test Coverage

**Test Files Located:** `/Users/project/Tests/UserProfileViewModelTests.swift`  
**Test Count:** 12 test methods  
**Coverage Assessment:** MEDIUM

## 5. CONTEXT ACQUISITION TRACE

### Files Read
1. `/Users/project/App/ViewModels/UserProfileViewModel.swift` - Target
2. `/Users/project/Models/User.swift` - Type dependency
3. `/Users/project/Networking/UserRepository.swift` - Injected dependency
... (12 files total)

### Dynamic Rules Applied
- ✅ Rule 1: Type Reference Resolution → 4 types resolved
- ✅ Rule 2: Protocol Conformance Chain → 2 protocols analyzed
- ✅ Rule 7: Test Coverage Mapping → TESTS_FOUND
- ✅ Rule 8: Extension Discovery → 1 extension found

## 6. RISK ASSESSMENT

### Overall Risk Level: MEDIUM

**Risk Calculation:**
```
- Public API changes: NO → +0
- Missing test coverage: PARTIAL → +MEDIUM
- Strict concurrency disabled: YES → +MEDIUM
TOTAL RISK: MEDIUM
```

### Missing Artifacts
| Artifact | Severity | Impact |
|----------|----------|--------|
| CODE_TEMPLATES.md | LOW | Will infer from existing code |
| Comprehensive test coverage | MEDIUM | Edge cases untested |

### Assumptions Made

⚠️ **ASSUMPTION:** UserRepository is thread-safe  
**Rationale:** No actor isolation detected  
**Risk if Wrong:** Potential data races

### High-Risk Areas
1. **Completion Handler Migration:** 5 methods need conversion
2. **Force Unwraps:** 7 detected - need careful replacement
3. **Missing @MainActor:** ViewModel touches UI without isolation

## 7. COMPLETENESS VERIFICATION

```
PROJECT FOUNDATION:
  ✅ README.md located and read
  ✅ Swift version confirmed: 6.3
  ⚠️ Strict concurrency: DISABLED
  ✅ Architecture identified: MVVM

TARGET ANALYSIS:
  ✅ Target file(s) read: 1
  ✅ Type inventory completed
  ✅ Concurrency characteristics analyzed

DEPENDENCY MAPPING:
  ✅ Type dependencies resolved (depth 2)
  ✅ Protocol conformances identified: 2
  ✅ Call sites mapped: 3 consumers
  ✅ Test files located: YES

COMPLETENESS GATES:
  ✅ All referenced types defined
  ✅ All protocols defined
  ⚠️ Concurrency types verified as Sendable (MISSING)
  ✅ Test coverage assessed: MEDIUM
```

### Gaps Requiring User Input

**QUESTION 1:** Is NSObject inheritance required for Objective-C interop?  
**Options:**
1. Yes - preserve inheritance
2. No - can remove and use pure Swift

**Recommended Action:** Verify if any Objective-C code calls this ViewModel

## 8. PHASE 2 READINESS

### Input for `/analyze-context`

This Context Summary provides Phase 2 with:
- ✅ Complete target file inventory (1 file)
- ✅ Type and dependency map (4 types, 2 protocols)
- ✅ Concurrency safety baseline
- ✅ Risk assessment (MEDIUM risk)
- ✅ Test coverage status (MEDIUM)

### Recommended Next Steps

1. **User Review:** Review this Context Summary
2. **Gap Resolution:** Clarify NSObject inheritance (Question 1)
3. **Approval Gate:** Confirm readiness
4. **Phase 2 Invocation:** Run `/analyze-context`

### Blockers

✅ No blockers - ready to proceed to Phase 2.

---

**END OF CONTEXT SUMMARY**
```

---

## Key Takeaways

This example demonstrates:
1. ✅ Complete YAML frontmatter
2. ✅ All 8 mandatory sections populated
3. ✅ Risk assessment with clear calculation
4. ✅ Documented assumptions with rationale
5. ✅ User clarification questions
6. ✅ Realistic completion status (mix of ✅, ⚠️)
7. ✅ Clear Phase 2 handoff

**File Location:** `.agent/artifacts/context_summary.md`

---

# EXECUTION WORKFLOW

## Phase 1: Mandatory Collection
```
EXECUTE IN ORDER:
1. Read target file(s) specified by user
2. Scan project root for Section 1 artifacts
3. Document found artifacts
4. Document missing artifacts with justification
5. Create initial Context Summary Document
```

## Phase 2: Static Code Analysis
```
FOR target file(s):
  1. Parse import statements
     → IF custom module THEN request definition
  
  2. Parse type declarations
     → APPLY Rule 1 (Type Reference Resolution)
  
  3. Parse protocol conformances
     → APPLY Rule 2 (Protocol Conformance Chain)
  
  4. Parse class inheritance
     → APPLY Rule 4 (Inheritance Hierarchy Traversal)
  
  5. Check access modifiers
     → IF public/open THEN APPLY Rule 3 (Public API Impact)
```

## Phase 3: Dynamic Dependency Resolution
```
SET depth_limit = 3
SET current_depth = 0

WHILE new types discovered AND current_depth < depth_limit:
  FOR EACH new_type IN discovered_types:
    1. Determine type characteristics
    2. Apply relevant rules (1-16) based on characteristics
    3. Request additional context
    4. Add to context collection
  
  INCREMENT current_depth
  
IF depth_limit reached:
  DOCUMENT: "Stopped at depth {depth_limit} to prevent infinite recursion"
```

## Phase 4: Completeness Verification
```
BEFORE proceeding to Phase 2 (Code Understanding):

VERIFY CHECKLIST:
  ✅ All referenced types have definitions
  ✅ All protocols have definitions
  ✅ All public API call sites mapped
  ✅ Test files identified (or absence noted as HIGH RISK)
  ✅ Concurrency types verified as Sendable

IF checklist incomplete:
  1. Document gaps in Context Summary
  2. Assess risk level for each gap (HIGH/MEDIUM/LOW)
  3. DECISION POINT:
     - Request missing items from user (if critical)
     - Proceed with caution (if non-critical)
     - HALT (if gaps block refactoring safely)
```

---

# IDE TOOL USAGE PATTERNS

## File Acquisition Strategies

### Strategy A: Direct Path Access
```
USE: view_file tool
WHEN: Exact file path is known
SYNTAX: view_file("/absolute/path/to/file.swift")
```

### Strategy B: Symbol-Based Search
```
USE: grep_search or codebase_search
WHEN: File path unknown, searching by type/function name
PATTERN: "class UserRepository" or "struct User"
THEN: view_file on discovered file path
```

### Strategy C: Pattern-Based Discovery
```
USE: find_by_name tool
WHEN: Searching by naming convention
PATTERNS:
  - "*ViewModel.swift" (all ViewModels)
  - "*Tests.swift" (all test files)
  - "*+Extension.swift" (extensions)
FILTER: By directory, extension, or max depth
```

## Call Site Discovery

### Function/Method Call Sites
```
TOOL: grep_search
PATTERN: "functionName("
OPTIONS:
  - MatchPerLine: true
  - CaseInsensitive: false
PROCESS: Iterate results → request each calling file
```

### Type Usage Sites
```
TOOL: grep_search
PATTERN: "TypeName" (with word boundary)
CHECK LOCATIONS:
  - Import statements
  - Property declarations
  - Parameter types
  - Return types
```

## Protocol Adopter Search
```
TOOL: grep_search
PATTERNS:
  - ": ProtocolName"
  - ", ProtocolName"
SCOPE: Entire codebase
FILTER: Include = ["*.swift"]
```

## Extension Search
```
TOOL: grep_search
PATTERN: "extension {TargetTypeName}"
SCOPE: All Swift files
PURPOSE: Locate functionality split across files
```

---

# CONTEXT PRIORITIZATION

When context size exceeds practical limits, collect in this priority order:

## Priority 1: CRITICAL (Always Collect)
1. Target file(s) - complete contents
2. Direct type dependencies - types directly used
3. Public API call sites - if modifying public APIs
4. Protocol definitions - contracts target must satisfy
5. Test files - behavioral specifications

## Priority 2: IMPORTANT (Collect if Feasible)
1. Superclass definitions - inheritance contracts
2. Subclass definitions - downstream impact
3. Dependency injection sources - lifetime management
4. Related data models - domain understanding
5. Architecture documentation - design context

## Priority 3: SUPPLEMENTARY (Collect if Relevant)
1. Similar patterns - consistency guidance
2. Historical changes - evolution context (git log)
3. Feature documentation - business context
4. Performance data - optimization constraints

---

# MISSING CONTEXT PROTOCOLS

## Protocol A: Context Not Found
```
IF required context cannot be located after exhaustive search
THEN:
  1. Document gap in Context Summary under "Missing Artifacts"
  2. Note in Risk Assessment with HIGH/MEDIUM/LOW severity
  3. Request from user:
     "Cannot locate definition for {Symbol}. Please provide:
      - File path if it exists
      - Confirmation if intentionally external/missing"
  4. HALT until user responds (do NOT assume or fabricate)
```

## Protocol B: Ambiguous Context
```
IF multiple definitions found for same symbol
THEN:
  1. List ALL discovered locations with file paths
  2. Request from user:
     "Found multiple definitions of {Symbol}:
      - {path1} (line {N})
      - {path2} (line {M})
      Which definition should be considered canonical?"
  3. WAIT for user clarification before proceeding
```

## Protocol C: Incomplete Context
```
IF partial understanding due to missing artifacts
THEN:
  1. Proceed with available context
  2. Document limitations in Context Summary:
     "⚠️ Limited Context: {description of gaps}"
  3. Prefix assumptions with "ASSUMPTION:"
     Example: "ASSUMPTION: UserRepository conforms to Sendable"
  4. Flag affected areas as HIGH RISK in Refactor Plan
```

---

# PHASE COMPLETION PROTOCOL

## Pre-Output Validation

Before generating Context Summary, verify:

```
CRITICAL CHECKS:
  ✅ All target files successfully read
  ✅ All custom types resolved or documented as missing
  ✅ All protocol definitions located
  ✅ Depth limit respected (max 3 levels)
  ✅ No tool errors or failures during gathering

QUALITY CHECKS:
  ✅ Risk assessment has rationale
  ✅ All assumptions documented with "ASSUMPTION:" prefix
  ✅ Missing artifacts cataloged with severity
  ✅ Test coverage status determined
  ✅ Completeness checklist filled (no empty checkboxes)
```

---

## Output Generation Steps

```
STEP 1: Populate Context Summary Template
  - Use template from OUTPUT SPECIFICATION section
  - Fill all mandatory sections (1-8)
  - Replace all {placeholders} with actual values
  - Ensure no "TODO" or "FILL_IN" markers remain

STEP 2: Validate Against Schema
  - Check YAML frontmatter completeness
  - Verify all 8 sections present
  - Confirm all mandatory fields have values
  - Validate file paths are absolute

STEP 3: Save Output File
  - Create directory: `.agent/artifacts/` (if not exists)
  - Write file: `.agent/artifacts/context_summary.md`
  - Confirm file written successfully

STEP 4: Present Summary to User
  - Display key metrics:
    * Files analyzed: {count}
    * Dependencies resolved: {count}
    * Risk level: {LOW|MEDIUM|HIGH}
    * Missing artifacts: {count}
    * Blockers: {count}
  
  - Show summary excerpt (first 50 lines)
  
  - Provide file location:
    "Context Summary saved to: .agent/artifacts/context_summary.md"
```

---

## User Approval Gate

```
PRESENT TO USER:

"Phase 1: Context Intake Complete ✅

Summary:
  - Target Files: {count}
  - Dependencies Mapped: {count}
  - Risk Level: {LOW|MEDIUM|HIGH}
  - Test Coverage: {HIGH|MEDIUM|LOW|NONE}
  - Missing Artifacts: {count}
  
{IF missing artifacts or assumptions:}
⚠️ Gaps Identified:
  - {list gaps}
  
{IF questions for user:}
❓ Questions Requiring Clarification:
  - {list questions}

{IF blockers:}
❌ Blockers Preventing Phase 2:
  - {list blockers}
  
📄 Full Context Summary: .agent/artifacts/context_summary.md

{IF no blockers:}
✅ Ready to proceed to Phase 2: Code Understanding

{IF blockers:}
⚠️ Please resolve blockers before proceeding.

---

Proceed to Phase 2? (yes/no/review)
  - yes: Continue to /analyze-context
  - no: Stop workflow
  - review: Open context_summary.md for detailed review
"

WAIT FOR USER RESPONSE

IF user responds "yes":
  OUTPUT: "✅ Phase 1 Complete. Context Summary saved to .agent/artifacts/context_summary.md"
  OUTPUT: "👉 **NEXT STEP:** Run the following command to start Phase 2:"
  OUTPUT: "/analyze-context"
  HALT (Phase 1 complete)

IF user responds "no":
  ASK: "What would you like to revise or clarify?"
  WAIT FOR FEEDBACK
  ITERATE on gaps

IF user responds "review":
  OUTPUT: "Please review .agent/artifacts/context_summary.md"
  OUTPUT: "When ready, respond with 'yes' to proceed or 'no' to revise."
  WAIT FOR USER RESPONSE
```

---

## Phase 1 → Phase 2 Handoff

```
HANDOFF PROTOCOL:

Phase 1 (Context Intake) produces:
  FILE: .agent/artifacts/context_summary.md
  FORMAT: Markdown with YAML frontmatter
  CONTAINS:
    - User objectives
    - Project foundation
    - Target file analysis
    - Dependency map
    - Risk assessment
    - Completeness verification

Phase 2 (Code Understanding) consumes:
  INPUT: .agent/artifacts/context_summary.md
  READS:
    - target_files (from YAML frontmatter)
    - dependency map (Section 4)
    - risk assessment (Section 6)
    - test coverage status (Section 4.4)
  
  PRODUCES:
    - Logic & Dependency Report
    - Code Analysis Report
    - Detailed concurrency analysis

CRITICAL RULE:
  Phase 2 MUST NOT re-gather context.
  Phase 2 MUST consume Phase 1 output.
  Phase 2 MUST reference context_summary.md for all baseline data.
```

---

## Success Criteria

Phase 1 is considered COMPLETE when:

```
✅ Context Summary generated and saved
✅ All mandatory sections populated
✅ All target files analyzed
✅ Dependencies mapped (within depth limit)
✅ Risk assessment completed with rationale
✅ Test coverage status determined
✅ User approval obtained (or blockers documented)
✅ File location provided to user
✅ No tool errors or failures

THEN: Phase 1 COMPLETE → Ready for Phase 2
```

---

## Failure Handling

IF Phase 1 cannot complete:

```
SCENARIO 1: Critical context missing
  ACTION: Document in Missing Artifacts (Section 6)
  SEVERITY: Determine if blocking or acceptable gap
  IF BLOCKING:
    - Set blocker in Section 8
    - Request user provide missing context
    - HALT until resolved
  IF ACCEPTABLE:
    - Document assumption
    - Flag as HIGH RISK
    - Proceed with caution

SCENARIO 2: Tool failures (file not found, search errors)
  ACTION: 
    - Log error in Context Acquisition Trace (Section 5)
    - Attempt alternative tool/strategy
    - IF unresolvable:
      - Document as blocker
      - Request user assistance
      - HALT

SCENARIO 3: Ambiguous context (multiple definitions)
  ACTION:
    - List all discovered options
    - Request user clarification (Section 7)
    - WAIT for response
    - Update context with clarified choice

SCENARIO 4: Depth limit reached
  ACTION:
    - Document in Section 5: "Recursion stopped at depth 3"
    - List unresolved dependencies in Missing Artifacts
    - Assess if blocking:
      - IF critical types missing → BLOCKER
      - IF supplementary → ACCEPTABLE GAP
```

---

## Post-Completion Checklist

Before declaring Phase 1 complete, verify:

```
FILE SYSTEM:
  ✅ .agent/artifacts/ directory exists
  ✅ context_summary.md file created
  ✅ File is valid Markdown
  ✅ File size > 0 bytes

CONTENT VALIDATION:
  ✅ YAML frontmatter valid
  ✅ All 8 sections present
  ✅ No {placeholder} text remaining
  ✅ All file paths are absolute
  ✅ Risk level assigned (LOW/MEDIUM/HIGH)
  ✅ Scope assigned (SMALL/MEDIUM/LARGE)

USER COMMUNICATION:
  ✅ Summary presented to user
  ✅ File location provided
  ✅ Approval requested
  ✅ Response received and processed

PHASE TRANSITION:
  ✅ User approved OR blockers documented
  ✅ Next phase instructions provided
  ✅ Context Summary path communicated
  ✅ Phase 1 marked complete
```

---

# CRITICAL OPERATING PRINCIPLES

1. **Completeness Over Speed**: Thorough context > fast progression
2. **Explicit Over Assumed**: Request clarification > make assumptions
3. **Document Gaps Honestly**: Note limitations > hide uncertainties
4. **Depth Limits Prevent Cycles**: Max 3 levels of dependency traversal
5. **Priority Guides Triage**: Critical context always collected first

---

*This protocol is authoritative for Phase 1 execution. Adhere strictly to acquisition rules and completeness gates before advancing to Phase 2.*

*Last Updated: 2025-11-28*  
*Swift 6.3 Refactor Agent*  
*Version: 2.0 - Self-Contained Execution Model*

