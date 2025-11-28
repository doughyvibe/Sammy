# plan-refactor

**Workflow Phase**: Phase 4 (Refactor Plan Generation)  
**Target**: Swift 6.3 Strategic Refactoring Plan
**Input**: `.agent/artifacts/review_report.md` (from Phase 3)

---

## AGENT DIRECTIVE

Synthesize findings from the **Review Report** and **Context Summary** into a concrete, sequenced refactoring plan. Transform identified issues into executable change specifications with dependencies, risk assessment, and testing requirements.

**CRITICAL**: This plan requires explicit user approval before execution—NO code modifications permitted in this phase.

**Output**: Comprehensive refactoring roadmap with sequenced changes, testing strategy, and rollback procedures.

---

## INPUT PARAMETERS

### Required Inputs
```
1. Review Report Artifact:
   Path: .agent/artifacts/review_report.md
   Content: Prioritized findings, compliance scores, critical issues from Phase 3.

2. Context Summary Artifact:
   Path: .agent/artifacts/context_summary.md
   Content: Architecture, dependencies, and risk assessment from Phase 1.
```

### Optional Inputs
```
{user_priorities}   : Focus areas specified by user
                      Examples: "focus on concurrency only", "prioritize safety over modernization"
```

---

## EXECUTION PROTOCOL

### PHASE 1: Ingest Review Report

```
ACTION: READ .agent/artifacts/review_report.md

EXTRACT:
  - Critical Issues (Must Fix)
  - High Priority Issues (Should Fix)
  - Medium Priority Issues (Nice to Have)
  - Compliance Scorecard (Baseline metrics)

VALIDATE:
  - Ensure all critical issues from Phase 3 are addressed in the plan.
  - Verify that proposed changes align with the "Top Recommendations" from the report.
```

### PHASE 2: Issue Prioritization & Grouping

Transform review findings into prioritized change groups:

```
CATEGORIZE findings by severity (CRITICAL, HIGH, MEDIUM, LOW)
GROUP into 5-tier priority system:
  
  TIER 1 - FOUNDATION (Execute First):
    RATIONALE: Enables subsequent changes, establishes safety baseline
    PATTERNS:
      - Sendable conformances (unblocks concurrency changes)
      - Actor isolation for shared state (prevents data races)
      - Dependency injection improvements (enables testing)
    RISK IMPACT: Changes here affect entire codebase
  
  TIER 2 - SAFETY CRITICAL (Execute Second):
    RATIONALE: Prevents crashes and runtime errors
    PATTERNS:
      - Force unwrap elimination (crash prevention)
      - Error handling improvements (graceful degradation)
      - Retain cycle fixes (memory leak prevention)
    RISK IMPACT: High user impact if bugs introduced
  
  TIER 3 - MODERNIZATION (Execute Third):
    RATIONALE: Swift 6.3 best practices, code readability
    PATTERNS:
      - Completion handler → async/await (structured concurrency)
      - DispatchQueue → Task/TaskGroup (modern patterns)
      - ObservableObject → @Observable (iOS 17+ efficiency)
    RISK IMPACT: Moderate - affects async flow
  
  TIER 4 - CODE QUALITY (Execute Fourth):
    RATIONALE: Maintainability and testability improvements
    PATTERNS:
      - Function extraction (reduce complexity)
      - Naming improvements (clarity)
      - Documentation additions (understanding)
    RISK IMPACT: Low - mostly refactoring
  
  TIER 5 - POLISH (Execute Last):
    RATIONALE: Nice-to-have enhancements
    PATTERNS:
      - Type safety improvements (Any → generics)
      - Code organization (MARK sections)
      - Additional test coverage
    RISK IMPACT: Very low - optional improvements

APPLY user_priorities filter IF specified:
  IF user_priorities contains "concurrency only":
    INCLUDE only TIER 1, TIER 2, relevant TIER 3 items
  ELSE IF user_priorities contains "safety only":
    INCLUDE only TIER 2 items
  ELSE:
    INCLUDE all tiers
```

### PHASE 3: Dependency Sequencing

Order changes within each tier by dependencies:

```
FOR EACH tier IN [TIER 1, TIER 2, TIER 3, TIER 4, TIER 5]:
  FOR EACH change IN tier:
    IDENTIFY dependencies:
      - Type dependencies (must define types before using them)
      - Call dependencies (must update definitions before call sites)
      - Module dependencies (must update imports before usage)
    
    BUILD dependency graph
  
  TOPOLOGICAL SORT changes by dependency graph
  
  VALIDATE sequencing rules:
    ✅ Dependencies come before dependents
    ✅ Type changes come before usage changes
    ✅ Breaking changes are minimized or justified
    ✅ Each change is independently buildable
    ✅ Each change is independently testable
  
  IF cycles detected:
    IDENTIFY cycle components
    MERGE into single atomic change
    NOTE: "ATOMIC CHANGE (cannot be split due to circular dependency)"
```

### PHASE 4: Change Specification Generation

Create detailed specification for EACH change:

```
FOR EACH change IN sequenced_changes:
  ASSIGN change_id = "CHANGE-{zero_padded_number}"  // e.g., CHANGE-001
  
  CREATE change_specification {
    change_id: "CHANGE-XXX",
    type: "{category from classification}",
    title: "{concise descriptive title}",
    files_affected: ["relative/path/to/file1.swift", ...],
    dependencies: ["CHANGE-YYY", "CHANGE-ZZZ"] OR "NONE",
    risk_level: "LOW" | "MEDIUM" | "HIGH" | "CRITICAL",
    complexity: 1-10,  // 1=trivial, 10=very complex
    
    current_code: "{exact code snippet from review finding}",
    proposed_code: "{complete corrected implementation}",
    
    rationale: "{why this improves quality - reference Swift 6.3 benefits}",
    success_criteria_ref: "{criterion ID from 1-16}",
    
    backward_compatibility: {
      classification: "NON-BREAKING" | "BREAKING" | "DEPRECATION",
      impact_description: "{what breaks and why}",
      mitigation_strategy: "{how to minimize impact}" OR "N/A"
    },
    
    testing_impact: {
      existing_tests_to_update: ["test file paths"],
      new_tests_to_add: ["test descriptions"],
      validation_steps: ["step-by-step verification"]
    }
  }
  
  ADD change_specification to plan
```

#### Change Type Classification

```
CLASSIFY each change into type:
  
  TYPE: "Add Protocol Conformance"
    PATTERN: Adding Sendable, Codable, Equatable, etc.
    COMPLEXITY: Usually 1-3
    RISK: LOW (additive change)
  
  TYPE: "Add Isolation Annotation"
    PATTERN: Adding @MainActor, actor keyword
    COMPLEXITY: 3-5
    RISK: MEDIUM (may affect call sites)
  
  TYPE: "Modernize Concurrency Pattern"
    PATTERN: Completion handler → async/await
    COMPLEXITY: 5-8
    RISK: MEDIUM to HIGH (signature changes)
  
  TYPE: "Safe Optional Handling"
    PATTERN: Force unwrap → guard let, if let, throw
    COMPLEXITY: 2-4
    RISK: LOW to MEDIUM (behavior preservation critical)
  
  TYPE: "Extract Function"
    PATTERN: Reduce cyclomatic complexity
    COMPLEXITY: 4-7
    RISK: LOW (internal refactoring)
  
  TYPE: "Error Handling Improvement"
    PATTERN: try! → do-catch, Result type
    COMPLEXITY: 3-5
    RISK: MEDIUM (error propagation changes)
  
  TYPE: "Type Safety Enhancement"
    PATTERN: Any → Generic, AnyObject → Protocol
    COMPLEXITY: 4-6
    RISK: MEDIUM (type system changes)
  
  TYPE: "Documentation Addition"
    PATTERN: Add DocC comments, MARK sections
    COMPLEXITY: 1-2
    RISK: LOW (non-functional change)
```

#### Risk Level Assessment

```
CALCULATE risk_level:
  
  CRITICAL:
    - Affects public API with >10 call sites
    - Changes data race safety guarantees
    - Modifies business logic inadvertently
  
  HIGH:
    - Affects public API with 3-10 call sites
    - Changes async/await flow significantly
    - Requires test refactoring
    - No existing test coverage
  
  MEDIUM:
    - Affects internal APIs
    - Changes method signatures (1-2 call sites)
    - Moderate complexity (6-8)
  
  LOW:
    - Internal refactoring only
    - Additive changes (conformances)
    - Well-tested code paths
    - Low complexity (1-3)
```

### PHASE 5: Testing Strategy Formulation

Define comprehensive testing approach:

```
CREATE testing_strategy {
  
  pre_refactor_baseline: {
    actions: [
      "Run full test suite: `swift test`",
      "Verify clean build: `swift build`",
      "Confirm strict concurrency mode enabled",
      "Record baseline metrics: {test_count} tests passing, {warning_count} warnings"
    ],
    acceptance_criteria: "All tests pass, zero compiler errors"
  },
  
  per_change_validation: {
    actions: [
      "Build succeeds: `swift build`",
      "Affected tests pass (run subset)",
      "Zero new compiler warnings introduced",
      "Commit with descriptive message: '{CHANGE-ID}: {title}'"
    ],
    frequency: "After EACH change or change group"
  },
  
  new_tests_required: [
    FOR EACH change WHERE testing_impact.new_tests_to_add.length > 0:
      {
        change_id: "CHANGE-XXX",
        test_name: "{descriptive test name}",
        test_purpose: "{what behavior to verify}",
        example_code: "{sample test implementation}"
      }
  ],
  
  integration_validation: {
    actions: [
      "Full test suite: All {baseline_count}+ tests pass",
      "Manual UI flow test: {critical user flow}",
      "Performance check: No >10% regressions in async operations",
      "Concurrency check: Zero runtime data race warnings"
    ],
    timing: "After ALL changes complete"
  }
}
```

### PHASE 6: Rollback Plan Definition

Specify recovery procedures for high-risk changes:

```
FOR EACH change WHERE risk_level IN ["HIGH", "CRITICAL"]:
  CREATE rollback_procedure {
    change_id: "CHANGE-XXX",
    
    rollback_steps: [
      "git revert {commit_hash}",
      "swift build  # verify compilation",
      "swift test   # verify baseline restored",
      "{additional_steps if needed}"
    ],
    
    failure_indicators: [
      "{specific symptom that suggests failure}",
      "{test failure pattern}",
      "{runtime behavior}",
      "{performance degradation}"
    ],
    
    mitigation_strategy: "{how to prevent need for rollback}",
    
    point_of_no_return: "{at what stage rollback becomes difficult}"
  }
```

### PHASE 7: Backward Compatibility Analysis

Identify and document breaking changes:

```
EXTRACT all changes WHERE backward_compatibility.classification == "BREAKING"

FOR EACH breaking_change:
  CREATE compatibility_decision {
    change_id: "CHANGE-XXX",
    what_breaks: "{API signature, behavior, return type}",
    affected_scope: "{number of call sites, modules, external consumers}",
    
    migration_options: [
      {
        option: "A",
        description: "Deprecate old API, keep both for {N} releases",
        pros: ["Gradual migration", "No immediate break"],
        cons: ["Maintenance burden", "Code duplication"],
        recommendation_level: "RECOMMENDED" | "OPTIONAL"
      },
      {
        option: "B",
        description: "Force immediate migration (update all {N} call sites)",
        pros: ["Clean cut", "No legacy code"],
        cons: ["High impact", "Risky"],
        recommendation_level: "RECOMMENDED" | "OPTIONAL"
      }
    ],
    
    user_decision_required: true,
    recommended_option: "{A or B with justification}"
  }
  
  ADD to user_approval_section
```

---

## OPERATIONAL CONSTRAINTS

### Forbidden Actions
```
❌ NEVER include new features (only refactor existing functionality)
❌ NEVER break public APIs without explicit user permission
❌ NEVER create big-bang changes (limit to 1-3 files per change)
❌ NEVER modify business logic behavior
❌ NEVER assume user intent or requirements
❌ NEVER skip dependency sequencing validation
```

### Required Actions
```
✅ ALWAYS sequence changes in dependency order
✅ ALWAYS include complexity ratings (1-10 scale)
✅ ALWAYS align with success criteria (reference #1-16)
✅ ALWAYS require user approval before execution
✅ ALWAYS provide rollback procedures for high-risk changes
✅ ALWAYS specify testing requirements explicitly
```

---

## OUTPUT SPECIFICATION

Generate comprehensive refactor plan in Markdown format:

### Plan Template

```markdown
# Refactor Plan - {TargetFiles}

**Generated**: {ISO 8601 timestamp}  
**Target**: {comma-separated file paths}  
**Input Source**: Review Report ({review_date})
**Context Source**: Context Summary ({analysis_date})

---

## Executive Summary

**Goals**:
- {Specific measurable goal 1}
- {Specific measurable goal 2}
- {Specific measurable goal 3}
- {General improvement goal}

**Non-Goals**:
- No new features or functionality
- No changes to business logic behavior
- No UI/UX changes (unless explicitly specified)

**Expected Benefits**:
- ✅ {Quantified benefit 1 - e.g., "Zero data race warnings"}
- ✅ {Quantified benefit 2 - e.g., "80% reduction in force unwraps"}
- ✅ {Quantified benefit 3 - e.g., "85%+ success criteria compliance"}

**Overall Risk**: {LOW | MEDIUM | HIGH | CRITICAL}
- {X} breaking changes ({mitigation strategy})
- {Y} high-complexity changes (complexity >7)
- {Test coverage assessment}

---

## Sequenced Changes (in dependency order)

### CHANGE-001: {Descriptive Title}

**Type**: {Change type from classification}  
**Files Affected**: 
- `relative/path/to/File1.swift`
- `relative/path/to/File2.swift` (if multi-file change)

**Dependencies**: {CHANGE-XXX, CHANGE-YYY} OR NONE

**Risk Level**: {LOW | MEDIUM | HIGH | CRITICAL}  
**Complexity**: {N}/10

**Current Code**:
```swift
{exact code snippet showing problematic pattern}
```

**Proposed Change**:
```swift
{complete corrected implementation with comments if complex}
```

**Rationale**: {Clear explanation of why this improves code quality, references Swift 6.3 features, cites specific benefits}

**Success Criteria Reference**: #{criterion_id} ({criterion_name})

**Backward Compatibility**: {NON-BREAKING | BREAKING | DEPRECATION}
{If BREAKING or DEPRECATION:}
- **Impact**: {Description of what breaks}
- **Affected Scope**: {Number of call sites or modules}
- **Mitigation**: {Strategy to minimize impact}

**Testing Impact**:
- ✅ {Existing test updates needed}
- ✅ {New tests to add}
- ✅ {Integration test considerations}

**Validation Steps**:
1. Build succeeds without errors
2. {Specific test passes}
3. {Runtime behavior verification}
4. {Performance check if applicable}

---

{Repeat for EACH change in sequence}

---

## Change Summary by Priority

**TIER 1 - Foundation** ({count} changes):
- CHANGE-001: {Brief description}
- CHANGE-002: {Brief description}
- ...

**TIER 2 - Safety Critical** ({count} changes):
- CHANGE-NNN: {Brief description}
- ...

**TIER 3 - Modernization** ({count} changes):
- CHANGE-MMM: {Brief description}
- ...

**TIER 4 - Code Quality** ({count} changes):
- CHANGE-KKK: {Brief description}
- ...

**TIER 5 - Polish** ({count} changes):
- CHANGE-JJJ: {Brief description}
- ...

**Total Changes**: {total_count}  
**Estimated Effort**: {low_estimate}-{high_estimate} hours  
**Files Modified**: {file_count}  
**Files Added**: {new_file_count} (if any)

---

## Swift 6.3 Modernization Checklist

TIER 1 Checklist:
- [ ] CHANGE-001: {Title}
- [ ] CHANGE-002: {Title}
- ...

TIER 2 Checklist:
- [ ] CHANGE-NNN: {Title}
- ...

{Continue for all tiers}

---

## Backward Compatibility Strategy

### Breaking Changes (User Decision Required)

**CHANGE-XXX**: {Method/API name} signature change

**What Breaks**: {Specific API elements that change}
**Affected Scope**: {N call sites in M files}

**Migration Options**:

**Option A: Gradual Migration (Recommended)**
- Approach: Deprecate old API, maintain both for 1-2 releases
- Implementation:
  ```swift
  // New async version
  {new_api}
  
  // Deprecated wrapper for compatibility
  @available(*, deprecated, message: "{migration message}")
  {old_api_wrapper}
  ```
- **Pros**: {List advantages}
- **Cons**: {List disadvantages}

**Option B: Immediate Migration**
- Approach: Update all {N} call sites immediately
- Impact: {Breaking change details}
- **Pros**: {List advantages}
- **Cons**: {List disadvantages}

**Recommended**: Option {A or B}  
**Rationale**: {Why this option is better for this specific case}

**🚨 User Decision Required**: Choose Option A or Option B

---

{Repeat for each breaking change}

### Non-Breaking Changes
All other changes ({count} changes) are non-breaking:
- Additive changes (protocol conformances)
- Internal refactoring
- Documentation additions

---

## Testing Strategy

### Pre-Refactor Baseline
**Actions**:
1. Run full test suite: `swift test`
2. Verify clean build: `swift build`
3. Confirm Swift 6.3 strict concurrency enabled in build settings
4. Record baseline: {N} tests passing, {M} compiler warnings

**Acceptance Criteria**: All tests pass, zero compiler errors

---

### Per-Change Validation Protocol
**After Each Change Group**:
1. Build succeeds: `swift build`
2. Run affected test subset: `swift test --filter {TestClass}`
3. Verify zero new compiler warnings
4. Git commit: `git commit -m "{CHANGE-ID}: {title}"`

**Halt Condition**: If any step fails, rollback change before proceeding

---

### New Tests to Add

#### Test 1: {Test Name} (CHANGE-XXX)
- **Purpose**: {What behavior to verify}
- **Test Type**: Unit | Integration | UI
- **Implementation**:
  ```swift
  @Test func {testMethodName}() async throws {
      // Arrange
      {setup_code}
      
      // Act
      {action_code}
      
      // Assert
      {assertion_code}
  }
  ```

{Repeat for each new test}

---

### Integration Validation (Post-All-Changes)
**Final Verification**:
1. Full test suite: All {N}+ tests pass (including new tests)
2. Manual UI flow test: {Critical user journey description}
3. Performance benchmark: No >10% regression in {key_metric}
4. Concurrency verification: Zero runtime data race warnings
5. Static analysis: SwiftLint passes (if configured)

**Acceptance Criteria**: All checks pass before considering refactor complete

---

## Rollback Plan

### High-Risk Changes (Detailed Recovery Procedures)

#### CHANGE-XXX: {Title} [RISK: {HIGH|CRITICAL}]

**Rollback Steps**:
```bash
# Revert specific commit
git revert {commit_hash}

# Verify build restored
swift build

# Verify tests restored
swift test

# {Additional recovery steps if needed}
```

**Failure Indicators** (When to Rollback):
- {Specific symptom 1 - e.g., "Tests fail with async timeout"}
- {Specific symptom 2 - e.g., "Runtime crash in production"}
- {Specific symptom 3 - e.g., "Deadlock in UI interactions"}

**Mitigation Strategy** (Prevent Rollback Need):
- {Preventive action 1}
- {Preventive action 2}

**Point of No Return**: {Description of when rollback becomes difficult}

---

{Repeat for each high-risk change}

### Emergency Full Rollback
```bash
# Return to pre-refactor state
git reset --hard {pre_refactor_commit_hash}

# Verify baseline
swift test
```

---

## Estimated Timeline

**Phase 4 (This Plan)**: {X} hours - Review and approval  
**Phase 5 (Execution)**: {Y}-{Z} hours over {N} sessions  
**Phase 6 (Validation)**: {V} hours - Comprehensive testing  
**Phase 7 (Documentation)**: {W} hours - Summary and walkthrough

**Total Estimated Effort**: {total_low}-{total_high} hours

**Recommended Execution Strategy**:
- [ ] Option A: All changes in single session (requires {hours} continuous time)
- [x] **Option B: Incremental execution over {N} sessions** (Recommended)
  - Session 1: TIER 1 changes ({duration})
  - Session 2: TIER 2 changes ({duration})
  - Session 3: TIER 3-5 changes ({duration})

---

## 🚨 USER APPROVAL REQUIRED

**Before proceeding to Phase 5 (Execution), please confirm**:

### 1. Plan Approval
- [ ] I approve this refactor plan

### 2. Backward Compatibility Decisions
FOR EACH breaking change:
- [ ] **CHANGE-XXX**: Choose Option ____ (A or B)
- [ ] **CHANGE-YYY**: Choose Option ____ (A or B)

### 3. Scope Modifications (Optional)
- [ ] No modifications needed - proceed as planned
- [ ] Add these changes: ____________
- [ ] Remove these changes: ____________
- [ ] Modify these changes: ____________

### 4. Execution Preference
- [ ] All at once (single session)
- [ ] Incremental with reviews (recommended)
- [ ] Custom schedule: ____________

### 5. Additional Requirements
- [ ] Code review required after: {specify milestone}
- [ ] Demo required after: {specify milestone}
- [ ] Stakeholder notification needed: {specify who}

---

**To Approve**: Respond with "approved" and compatibility decisions  
**To Modify**: Respond with requested changes  
**To Reject**: Respond with reasons for rejection

---

*This plan was generated from review-code.md findings and analyze-context.md data following the Swift 6.3 refactoring workflow. No code has been modified. Execution (Phase 5) requires explicit user approval.*
```

---

## PHASE COMPLETION PROTOCOL

1. **Generate Output**: Save plan to `.agent/artifacts/refactor_plan.md`.
2. **Validate**: Ensure all sections are populated and user approval section is clear.
3. **Present to User**:
   ```
   ✅ Phase 4 Complete. Refactor Plan generated.
   
   **Total Changes**: {Count}
   **Estimated Effort**: {Hours}
   **Breaking Changes**: {Count}
   
   👉 **NEXT STEP:** Review the plan in `.agent/artifacts/refactor_plan.md` and provide approval to proceed to execute`/apply-refactor`.
   ```

---

*Last Updated: 2025-11-28*
*Swift 6.3 Refactor Agent - Phase 4*
