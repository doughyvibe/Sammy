# Swift 6.3 Refactor Agent Workflow

**Document Purpose**: Sequential 7-phase execution protocol for systematic code refactoring  
**Authority**: All refactoring operations must follow this workflow sequence

---

## WORKFLOW OVERVIEW

```
EXECUTION MODEL: Sequential 7-phase pipeline

PHASE TRANSITION RULES:
  - Phases execute in strict order (1 → 2 → 3 → 4 → 5 → 6 → 7)
  - Each phase must complete successfully before next phase
  - Phase 4 requires explicit user approval before Phase 5
  - No code edits permitted until Phase 5
  - Validation failure in Phase 6 halts workflow

CRITICAL RULE: NEVER skip phases or execute out of order
```

### Phase Sequence

| # | Phase Name | Purpose | Code Edits | Output |
|---|------------|---------|------------|--------|
| 1 | Context Intake | Gather requirements | ❌ NO | Context Summary |
| 2 | Code Understanding | Analyze implementation | ❌ NO | Code Analysis Report |
| 3 | Impact & Dependency Analysis | Map blast radius | ❌ NO | Dependency Map |
| 4 | Refactor Plan | Design changes | ❌ NO | Refactor Plan (needs approval) |
| 5 | Refactor Execution | Implement changes | ✅ YES | Refactored Code Files |
| 6 | Validation | Verify quality | ✅ Fixes only | Validation Report |
| 7 | Report & Documentation | Document work | ✅ Docs only | Summary Report |

---

## PHASE 1: Context Intake

**Objective**: Gather comprehensive project context to establish refactoring foundation

### Input Requirements

```
REQUIRED INPUTS:
  target_files      : Absolute path(s) to Swift file(s) requiring refactoring
  user_request      : Specific refactoring goals, pain points, improvements
  swift_version     : Confirm project uses Swift 6.3

CONTEXT DOCUMENTS (gather if exist):
  - README.md
  - PRODUCT_SPEC.md
  - ENGINEERING_STANDARDS.md
  - CODE_TEMPLATES.md
  - .cursorrules or project guidelines

IDE STATE:
  - Open files in IDE
  - Recent conversation history
```

### Execution Protocol

```
STEP 1: Validate Inputs
  VERIFY target_files exist and are accessible
  CONFIRM user_request is clear and specific
  
  IF user_request is ambiguous:
    ASK clarifying questions
    WAIT for response
    RESTART Phase 1 with clarified requirements

STEP 2: Gather Governance Documents
  SEARCH for:
    - README.md (project overview)
    - ENGINEERING_STANDARDS.md (coding standards)
    - CODE_TEMPLATES.md (patterns, templates)
    - PRODUCT_SPEC.md (business requirements)
  
  FOR EACH document found:
    READ and EXTRACT relevant constraints
    STORE location for later reference

STEP 3: Assess Swift 6.3 Adoption
  CHECK for:
    - Strict concurrency mode enabled/disabled
    - async/await usage patterns
    - Actor usage
    - Sendable conformances
    - @MainActor annotations
  
  RECORD current adoption level

STEP 4: Define Refactoring Scope
  FROM user_request, EXTRACT:
    - Target files and components
    - Features affected
    - Known constraints or limitations
  
  CLASSIFY scope:
    - SMALL: 1-3 files, single component
    - MEDIUM: 4-10 files, multiple components
    - LARGE: >10 files, architectural changes

STEP 5: Perform Initial Risk Assessment
  IDENTIFY high-level risks:
    - Breaking changes to public APIs
    - Data migration requirements
    - Complex dependency chains
    - Areas without test coverage
  
  ASSIGN overall risk level: LOW, MEDIUM, HIGH
```

### Output Specification

Generate **Context Summary Document**:

```markdown
# Refactoring Context Summary

**Project**: {ProjectName}
**Target Files**: {list of absolute paths}
**Swift Version**: 6.3
**Scope**: {SMALL | MEDIUM | LARGE}
**Overall Risk**: {LOW | MEDIUM | HIGH}

## User Objectives
{bullet list of stated refactoring goals}

## Success Criteria
{user-defined measures of success}

## Project Architecture
- Pattern: {MVVM | MVC | VIPER | Composable Architecture | etc.}
- Key characteristics: {list}

## Governance Documents Located
- README.md: {path or NOT FOUND}
- ENGINEERING_STANDARDS.md: {path or NOT FOUND}
- CODE_TEMPLATES.md: {path or NOT FOUND}
- PRODUCT_SPEC.md: {path or NOT FOUND}

## Swift 6.3 Feature Adoption
- Strict concurrency mode: {ENABLED | DISABLED}
- Async/await usage: {WIDESPREAD | PARTIAL | MINIMAL}
- Actor usage: {PRESENT | ABSENT}
- Sendable conformance: {EXPLICIT | INFERRED | MISSING}

## Known Constraints
{list of limitations from governance docs or user}

## Initial Risk Assessment
{HIGH_LEVEL risks identified}
```

### Guardrails

```
FORBIDDEN ACTIONS:
  ❌ NO code edits during this phase
  ❌ NO assumptions about code quality without evidence
  ❌ NO skipping documentation review

REQUIRED ACTIONS:
  ✅ MUST ask clarifying questions if scope ambiguous
  ✅ MUST identify all governance documents
  ✅ MUST confirm target files exist
  ✅ MUST document Swift 6.3 adoption status
```

### Phase Completion Criteria

```
PHASE 1 COMPLETE IF:
  ✅ Context Summary Document generated
  ✅ All target files confirmed accessible
  ✅ User objectives clearly documented
  ✅ Governance documents located (or noted as absent)
  ✅ Initial risk assessment completed

THEN: Proceed to Phase 2
```

---

## PHASE 2: Code Understanding

**Objective**: Perform deep analysis of target code to understand current implementation

### Input Requirements

```
REQUIRED INPUTS:
  - Context Summary Document (from Phase 1)
  - Target Swift file(s) identified in Phase 1

SUPPORTING FILES (gather for context):
  - Protocol definitions
  - Model/Entity classes
  - View files (if refactoring ViewModels)
  - Service/Repository layers
  - Test files for target code
```

### Execution Protocol

```
STEP 1: Structural Analysis
  FOR EACH target file:
    INVENTORY:
      - Classes, structs, actors, enums
      - Protocol conformances
      - Inheritance hierarchies
      - Extension organization
    
    RECORD in analysis report

STEP 2: Swift 6.3 Concurrency Analysis
  EXAMINE:
    - Sendable conformance status (explicit vs inferred vs missing)
    - Actor usage and isolation boundaries
    - @MainActor annotations (presence and correctness)
    - Async/await vs completion handler patterns
    - Task and TaskGroup usage
    - Potential data races (nonisolated, isolated keywords)
  
  COMPILE concurrency safety assessment

STEP 3: Design Pattern Identification
  DETECT patterns:
    - Dependency injection mechanisms
    - State management approach
    - Observer patterns (Combine, NotificationCenter, @Published)
    - Singleton usage
    - Factory patterns
  
  DOCUMENT pattern catalog

STEP 4: Technical Debt Cataloging
  SCAN for code smells:
    - Force unwraps (!, try!, as!)
    - Implicitly unwrapped optionals
    - Legacy concurrency (DispatchQueue, semaphores)
    - Large classes (>500 LOC)
    - High cyclomatic complexity (>10)
    - Magic numbers/strings
    - Missing error handling (try?, swallowed errors)
  
  COUNT and LOCATE each instance

STEP 5: Test Coverage Assessment
  LOCATE test files
  ANALYZE:
    - Unit test coverage for target code
    - Integration test presence
    - Testing framework (XCTest, Swift Testing)
  
  ESTIMATE coverage percentage
```

### Output Specification

Generate **Code Analysis Report**:

```markdown
# Code Analysis Report

**Target Files Analyzed**: {count}
**Analysis Date**: {ISO 8601 timestamp}

## Structural Overview

### Type Inventory
- Classes: {count}
- Structs: {count}
- Actors: {count}
- Enums: {count}
- Protocols: {count}

### Key Types
{FOR EACH major type:}
- `{TypeName}` ({class|struct|actor})
  - Conforms to: {protocol list}
  - Inherits from: {superclass or N/A}
  - Extensions: {count}

## Swift 6.3 Concurrency Analysis

### Sendable Conformance Status
- Explicit Sendable types: {count}
- Inferred Sendable types: {count}
- Missing Sendable (needs adding): {count}

{FOR EACH missing Sendable type:}
- `{TypeName}` in `{file}:{line}`

### Actor Isolation
- Actors defined: {count}
- @MainActor classes: {count}
- Isolation boundaries: {count}

### Async/Await Adoption
- Async functions: {count}
- Completion handlers remaining: {count}
- DispatchQueue usage: {count} instances

### Data Race Risk Assessment
- Potential data races: {count}
- Unprotected global state: {count}

## Design Patterns Identified

### Dependency Injection
- Constructor injection: {PRESENT | ABSENT}
- Protocol abstractions: {count}
- Singleton usage: {count} ({list of singletons})

### State Management
- Pattern: {Combine | ObservableObject | Custom}
- @Published properties: {count}

## Technical Debt Catalog

### Force Unwraps
- Total count: {count}
- Locations: {file:line list}

### Optionals
- Implicitly Unwrapped Optionals: {count}
- Unnecessary optionals: {count}

### Legacy Patterns
- Completion handlers: {count}
- DispatchQueue usage: {count}
- GCD primitives: {count}

### Complexity Metrics
- Functions >50 lines: {count}
- Functions >80 lines: {count}
- Cyclomatic complexity >10: {count}

### Missing Error Handling
- try? (silenced errors): {count}
- Swallowed catch blocks: {count}

## Existing Test Coverage

### Test Files Located
{list of test file paths}

### Coverage Estimate
- Unit test coverage: {percentage}% (estimated)
- Integration tests: {PRESENT | ABSENT}
- Testing framework: {XCTest | Swift Testing | Mixed}
```

### Guardrails

```
FORBIDDEN ACTIONS:
  ❌ NO code edits during this phase
  ❌ NO jumping to solutions or proposals
  ❌ NO ignoring test files

REQUIRED ACTIONS:
  ✅ MUST use code navigation tools (view_file, view_code_item, codebase_search)
  ✅ MUST document all Sendable conformance gaps
  ✅ MUST identify all isolation boundaries
  ✅ MUST note compiler warnings/errors
```

### Phase Completion Criteria

```
PHASE 2 COMPLETE IF:
  ✅ Code Analysis Report generated
  ✅ All target files analyzed
  ✅ Concurrency safety assessment completed
  ✅ Technical debt cataloged
  ✅ Test coverage assessed

THEN: Proceed to Phase 3
```

---

## PHASE 3: Impact & Dependency Analysis

**Objective**: Map dependencies to understand blast radius of proposed changes

### Input Requirements

```
REQUIRED INPUTS:
  - Code Analysis Report (from Phase 2)
  - Target file(s) and their dependencies
  - Access to entire codebase for dependency mapping
  - Build configuration (conditional compilation understanding)
```

### Execution Protocol

```
STEP 1: Incoming Dependency Analysis
  USE grep_search and codebase_search
  
  FOR EACH public API in target files:
    FIND all usage locations
    CLASSIFY consumers:
      - Internal modules
      - Test files
      - External packages (if applicable)
    
    COUNT usage occurrences
  
  IDENTIFY:
    - Public API surface area
    - Protocol adopters
    - Inheritance hierarchies

STEP 2: Outgoing Dependency Analysis
  FOR EACH target file:
    EXTRACT imports
    IDENTIFY consumed services/modules
    MAP database/persistence usage
  
  CATALOG external dependencies:
    - Apple frameworks (Foundation, SwiftUI, etc.)
    - Third-party libraries
    - Internal modules

STEP 3: Concurrency Boundary Mapping
  TRACE data flow across isolation boundaries:
    - Where Sendable types are required
    - Actor isolation boundaries crossed
    - @MainActor propagation requirements
    - Potential data race locations
  
  FOR EACH boundary crossing:
    VERIFY type safety
    IDENTIFY missing Sendable conformances

STEP 4: Test Dependency Analysis
  IDENTIFY tests that will need updating:
    - Direct tests of target code
    - Integration tests spanning target code
    - Mock/stub dependencies
  
  ESTIMATE test update effort

STEP 5: Impact Scoring
  FOR EACH planned change:
    CALCULATE impact score:
      IF no public API changes:
        IMPACT = LOW
      ELSE IF public API changes with backward compatibility:
        IMPACT = MEDIUM
      ELSE IF breaking changes:
        IMPACT = HIGH
  
  GENERATE impact score matrix
```

### Output Specification

Generate **Dependency Map & Impact Assessment**:

```markdown
# Dependency Map & Impact Assessment

**Analysis Date**: {ISO 8601 timestamp}

## Incoming Dependencies (Consumers)

### Public API Surface
{FOR EACH public API:}
- `{signature}` in `{file}`
  - Usage count: {count}
  - Consumers: {list of files}
  - Risk if changed: {LOW | MEDIUM | HIGH}

### Protocol Adopters
{FOR EACH protocol:}
- `{ProtocolName}`: {count} conforming types
  - {list of conforming type names}

## Outgoing Dependencies (Consumed)

### Framework Dependencies
- Apple frameworks: {list}
- Third-party libraries: {list}
- Internal modules: {list}

### Service Dependencies
{FOR EACH service used:}
- `{ServiceName}`: {usage description}

### Persistence Dependencies
- Database: {CoreData | SwiftData | Realm | SQLite | None}
- UserDefaults usage: {YES | NO}
- Keychain usage: {YES | NO}

## Concurrency Boundary Analysis

### Sendable Requirements
{FOR EACH boundary crossing:}
- Location: `{file}:{line}`
- Type crossing: `{TypeName}`
- Status: {SENDABLE | NEEDS SENDABLE}

### Actor Isolation Boundaries
{FOR EACH actor boundary:}
- Actor: `{ActorName}`
- Crossing points: {count}
- Types involved: {list}

### @MainActor Propagation
- Classes requiring @MainActor: {list}
- Methods requiring @MainActor: {count}

## Test Dependency Chain

### Tests Requiring Updates
{FOR EACH test file:}
- `{TestFileName}`: {update reason}

### Mock/Stub Dependencies
- Mocks to update: {list}
- New mocks needed: {list}

## Impact Score Matrix

| Change Type | Files Affected | Impact Level | Risk |
|-------------|----------------|--------------|------|
| {change description} | {count} | {LOW\|MED\|HIGH} | {risk description} |

## Risk Register

### Identified Breaking Changes
{FOR EACH breaking change:}
- **Change**: {description}
- **Affected**: {list of consumers}
- **Mitigation**: {deprecation strategy}

### Areas Without Test Coverage
{list of untested code paths}

### Thread Safety Concerns
{list of concurrency risks}

### Performance Implications
{list of potential performance impacts}
```

### Guardrails

```
FORBIDDEN ACTIONS:
  ❌ NO code edits during this phase
  ❌ NO dismissing low-level dependencies
  ❌ NO skipping Sendable conformance checks

REQUIRED ACTIONS:
  ✅ MUST use grep_search and codebase_search
  ✅ MUST verify every public API for external usage
  ✅ MUST check for protocol witnesses affected
  ✅ MUST identify all actor isolation boundary crossings
  ✅ MUST flag deprecated Swift API dependencies
```

### Phase Completion Criteria

```
PHASE 3 COMPLETE IF:
  ✅ Dependency Map generated
  ✅ All incoming dependencies identified
  ✅ All outgoing dependencies cataloged
  ✅ Concurrency boundaries mapped
  ✅ Impact score matrix completed
  ✅ Risk register populated

THEN: Proceed to Phase 4
```

---

## PHASE 4: Refactor Plan

**Objective**: Synthesize findings into concrete, sequenced refactoring plan

### Input Requirements

```
REQUIRED INPUTS:
  - Context Summary Document (Phase 1)
  - Code Analysis Report (Phase 2)
  - Dependency Map & Impact Assessment (Phase 3)
  - Project governance documents
```

### Execution Protocol

```
STEP 1: Synthesize Requirements
  COMBINE insights from Phases 1-3
  ALIGN with user objectives from Phase 1
  IDENTIFY all changes needed

STEP 2: Categorize Changes
  GROUP changes by type:
    - Concurrency modernization (async/await, actors, Sendable)
    - Safety improvements (force unwrap elimination)
    - Code quality (refactoring, naming)
    - Testing enhancements
  
  FOR EACH change:
    ASSIGN:
      - Type of change
      - Files affected
      - Rationale
      - Swift 6.3 principle applied
      - Complexity rating (1-10)
      - Risk level (LOW | MEDIUM | HIGH)

STEP 3: Sequence Changes by Dependencies
  BUILD dependency graph:
    FOR EACH change:
      IDENTIFY prerequisites (what must be done first)
    
  SORT changes topologically:
    FOUNDATION changes first (Sendable conformances)
    DEPENDENT changes second (async/await conversions)
    REFINEMENT changes last (code quality)

STEP 4: Design Backward Compatibility Strategy
  FOR EACH breaking change:
    CREATE deprecation path:
      - Deprecate old API
      - Introduce new API
      - Bridge old to new for transition period
    
    WRITE migration guide section

STEP 5: Define Testing Strategy
  SPECIFY:
    - Which tests to update
    - New tests to add
    - Testing order (unit → integration → e2e)
  
  FOR EACH test update:
    EXPLAIN rationale

STEP 6: Create Rollback Plan
  FOR EACH high-risk change:
    DEFINE rollback procedure:
      - Git commands to revert
      - Rollback triggers (when to revert)
      - Rollback validation (how to verify)
```

### Output Specification

Generate **Comprehensive Refactor Plan**:

```markdown
# Refactor Plan

**Project**: {ProjectName}
**Target Files**: {count}
**Overall Risk**: {LOW | MEDIUM | HIGH}
**Estimated Complexity**: {1-10 rating}

## Executive Summary

### Goals
{bullet list of refactoring goals}

### Non-Goals
{bullet list of what we're NOT doing}

### Expected Benefits
- **Readability**: {description}
- **Safety**: {description}
- **Maintainability**: {description}
- **Performance**: {description}

### Overall Risk Assessment
{risk level with justification}

## Sequenced Changes

{FOR EACH change in dependency order:}

### CHANGE-{ID}: {Title}

**Files Affected**: {list}
**Type**: {Async Conversion | Add Sendable | Extract Method | etc.}
**Complexity**: {1-10}
**Risk**: {LOW | MEDIUM | HIGH}

**Rationale**:
{why this change improves code}

**Swift 6.3 Principle**:
{which Swift 6.3 best practice this addresses}

**Dependencies**:
{list of prerequisite change IDs or "None"}

**Implementation Details**:
{specific steps to execute this change}

---

## Swift 6.3 Modernization Tasks

### Async/Await Conversions
{list of completion handlers to convert}

### Sendable Conformances
{list of types needing Sendable}

### Actor Introductions
{list of classes to convert to actors}

### @MainActor Annotations
{list of UI-touching code needing annotation}

### Structured Concurrency Replacements
{list of DispatchQueue usage to replace}

## Backward Compatibility Strategy

{FOR EACH breaking change:}

### Breaking Change: {description}

**Deprecation Path**:
```swift
// Old API (deprecated)
@available(*, deprecated, message: "Use {newAPI}() instead")
func oldAPI() {
    // Bridge to new
}

// New API
func newAPI() async throws {
    // Modern implementation
}
```

**Migration Guide**:
{how consumers should update}

## Testing Strategy

### Tests to Update
{FOR EACH test file:}
- `{TestFileName}`: {update description}

### New Tests to Add
{FOR EACH new test:}
- `{TestName}`: {test purpose}

### Testing Order
1. Unit tests (verify individual changes)
2. Integration tests (verify interactions)
3. E2E tests (verify user flows)

## Rollback Plan

{FOR EACH high-risk change:}

### Rollback for CHANGE-{ID}

**Trigger**: {when to rollback}

**Procedure**:
```bash
git revert {commit_hash}
git push
```

**Validation**: {how to verify rollback success}

## User Approval Required

**CRITICAL**: This plan must be approved before Phase 5 execution.

Approval questions:
1. Do sequenced changes align with your objectives?
2. Is backward compatibility strategy acceptable?
3. Are high-risk changes justified?
4. May I proceed to Phase 5 (Refactor Execution)?
```

### Guardrails

```
FORBIDDEN ACTIONS:
  ❌ NO code edits during this phase
  ❌ NO vague descriptions (each change must be specific)
  ❌ NO ignoring governance document requirements
  ❌ NO plans that skip test updates

REQUIRED ACTIONS:
  ✅ MUST sequence changes in dependency order
  ✅ MUST align with ENGINEERING_STANDARDS.md
  ✅ MUST get user approval before Phase 5
  ✅ MUST call out Swift 6.3 strict concurrency deviations
  ✅ MUST include complexity ratings (1-10)
```

### Phase Completion Criteria

```
PHASE 4 COMPLETE IF:
  ✅ Refactor Plan generated
  ✅ All changes sequenced by dependencies
  ✅ Backward compatibility strategy defined
  ✅ Testing strategy specified
  ✅ Rollback plan created
  ✅ USER APPROVAL OBTAINED

THEN: Proceed to Phase 5
CRITICAL: MUST wait for explicit user approval
```

---

## PHASE 5: Refactor Execution

**Objective**: Systematically implement approved refactor plan

### Input Requirements

```
REQUIRED INPUTS:
  - Approved Refactor Plan (from Phase 4)
  - All target files identified in plan
  - Test files for validation
  - USER APPROVAL confirmation
```

### Execution Protocol

```
STEP 1: Pre-Execution Validation
  VERIFY user approval obtained
  CONFIRM all target files accessible
  RUN baseline tests (establish passing state)

STEP 2: Sequential Change Implementation
  FOR EACH change IN plan order:
    EXECUTE change following implementation details
    
    APPLY Swift 6.3 patterns:
      - @Sendable for closures crossing boundaries
      - @MainActor for UI-related classes/methods
      - nonisolated ONLY with justification
      - isolated parameters where appropriate
    
    USE modern Swift syntax:
      - if let shorthand
      - Primary associated types
      - some and any keywords correctly
    
    BUILD project:
      IF build fails:
        STOP execution
        REPORT error to user
        WAIT for guidance
    
    RUN relevant tests:
      IF tests fail:
        ROLLBACK change
        REPORT failure
        WAIT for guidance
    
    LOG change in execution log:
      - Change ID
      - Status (COMPLETED | FAILED | ROLLED BACK)
      - Timestamp
      - Any deviations from plan

STEP 3: Continuous Integration Checks
  AFTER each logical change group:
    VERIFY:
      - Zero compiler errors
      - Zero new warnings
      - All tests passing
      - No new force unwraps introduced
```

### Output Specification

Generate **Execution Log**:

```json
{
  "execution_start": "ISO 8601 timestamp",
  "execution_end": "ISO 8601 timestamp",
  "changes_completed": [
    {
      "change_id": "CHANGE-001",
      "status": "COMPLETED",
      "timestamp": "ISO 8601",
      "files_modified": ["path1", "path2"],
      "deviation_from_plan": "None",
      "build_status": "SUCCESS",
      "tests_passed": true
    }
  ],
  "changes_failed": [],
  "rollbacks_performed": []
}
```

**Refactored Code Files**: Updated Swift files with changes applied

### Guardrails

```
FORBIDDEN ACTIONS:
  ❌ NO deviating from approved plan without user consent
  ❌ NO making multiple unrelated changes simultaneously
  ❌ NO skipping intermediate build verification
  ❌ NO leaving commented-out code without documentation

REQUIRED ACTIONS:
  ✅ MUST follow exact sequence from Phase 4
  ✅ MUST build successfully after each change group
  ✅ MUST preserve existing functionality
  ✅ MUST add concurrency annotations properly
  ✅ MUST use modern Swift syntax
  ✅ MUST handle errors properly (no swallowing)
  ✅ MUST maintain consistent code style
```

### Phase Completion Criteria

```
PHASE 5 COMPLETE IF:
  ✅ All planned changes executed
  ✅ Execution log generated
  ✅ All builds successful
  ✅ All tests passing
  ✅ Code files updated and saved

THEN: Proceed to Phase 6
```

---

## PHASE 6: Validation

**Objective**: Verify refactored code meets all quality standards

*See `/tools/validate-changes.md` for complete validation protocol*

### Phase Completion Criteria

```
PHASE 6 COMPLETE IF:
  ✅ Validation Report generated
  ✅ All Gate 1 (Hard Requirements) passed
  ✅ At least 4/5 Gate 2 (Quality Targets) passed
  ✅ No blocking issues identified

IF VALIDATION FAILS:
  STOP workflow
  REPORT failures to user
  OFFER rollback options
  WAIT for user decision

THEN: Proceed to Phase 7
```

---

## PHASE 7: Report & Documentation

**Objective**: Document refactoring work and provide knowledge transfer

*See `/tools/explain-changes.md` for complete documentation protocol*

### Phase Completion Criteria

```
PHASE 7 COMPLETE IF:
  ✅ Refactoring Summary Report generated
  ✅ Migration Guide created (if breaking changes)
  ✅ Inline documentation updated
  ✅ Changelog entry added
  ✅ Knowledge base entry created

WORKFLOW COMPLETE: Refactoring successfully finished
```

---

## SWIFT 6.3 BEST PRACTICES REFERENCE

Apply these principles throughout all phases:

### Concurrency & Sendable
```
✅ All types crossing boundaries are Sendable
✅ @MainActor applied to UI-related types
✅ Actors used for shared mutable state
✅ Structured concurrency (Task, TaskGroup) over DispatchQueue
✅ Async/await over completion handlers
✅ Minimal global actors
```

### Type Safety
```
✅ some keyword for opaque return types
✅ any keyword for existential types
✅ Primary associated types in protocols
✅ Avoid force unwraps (!), force casts (as!)
✅ guard/if let for optional handling
```

### Modern Swift Syntax
```
✅ If-let shorthand: if let value { }
✅ Result builders for DSLs
✅ Property wrappers for cross-cutting concerns
✅ Trailing closure syntax
✅ Multi-line string literals
```

### Memory & Performance
```
✅ Value types (structs) over reference types (classes)
✅ Copy-on-write for large value types
✅ Lazy evaluation for expensive computations
✅ Weak/unowned to avoid retain cycles
✅ Profile before optimizing
```

### Code Organization
```
✅ Protocol-oriented design
✅ Single Responsibility Principle
✅ Dependency injection over singletons
✅ Extensions for protocol conformance
✅ Meaningful naming
```

### Error Handling
```
✅ Typed errors (specific Error conformances)
✅ Never silently swallow errors
✅ Propagate errors with throws
✅ Result type for composable errors
```

---

## CRITICAL WORKFLOW RULES

### Rule 1: Sequential Execution
```
PHASES MUST execute in order: 1 → 2 → 3 → 4 → 5 → 6 → 7
NO skipping phases
NO executing out of order
```

### Rule 2: No Code Edits Until Phase 5
```
PHASES 1-4: Read-only analysis
PHASE 5: Implementation begins
PHASE 6: Validation fixes only
PHASE 7: Documentation only
```

### Rule 3: User Approval Gate
```
AT END OF PHASE 4:
  PRESENT refactor plan to user
  WAIT for explicit approval
  DO NOT proceed to Phase 5 without approval
```

### Rule 4: Validation Failure Response
```
IF PHASE 6 VALIDATION FAILS:
  STOP workflow execution
  GENERATE failure report
  OFFER rollback options
  WAIT for user guidance
  DO NOT proceed to Phase 7
```

### Rule 5: Guardrail Compliance
```
RESPECT all ❌ FORBIDDEN and ✅ REQUIRED markers
NO exceptions without explicit user permission
WHEN uncertain, ASK user for clarification
```

---

## USAGE IN .cursorrules

To integrate this workflow in Cursor IDE:

```
When performing Swift code refactoring:

MUST follow 7-phase workflow:
  1. Context Intake (read-only)
  2. Code Understanding (read-only)
  3. Impact & Dependency Analysis (read-only)
  4. Refactor Plan (read-only, needs approval)
  5. Refactor Execution (code edits begin)
  6. Validation (verify quality)
  7. Report & Documentation (document work)

CRITICAL RULES:
  - NO skipping phases
  - NO code edits before Phase 5
  - MUST obtain user approval after Phase 4
  - MUST complete each phase successfully
  - MUST produce all required outputs
  - MUST respect all guardrails
  - MUST apply Swift 6.3 best practices
  - MUST validate against governance documents
```

---

**WORKFLOW END**

*This systematic workflow ensures safe, high-quality refactoring aligned with Swift 6.3 best practices. All phases must be completed in sequence with proper validation gates.*

*Last Updated: 2025-11-26*  
*Swift Version: 6.3*
