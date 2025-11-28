# validate-changes

**Workflow Phase**: Phase 6 (Comprehensive Validation)  
**Target**: Swift 6.3 Quality Gate Verification

---

## AGENT DIRECTIVE

Execute comprehensive validation of refactored code against strict quality gates. Verify technical correctness, test passage, compliance adherence, and objective achievement. **CRITICAL**: This is the final quality checkpoint before completion—ALL Gate 1 requirements MUST pass without exception.

**Output**: Detailed validation report with pass/fail status and actionable recommendations

---

## OUTPUT ARTIFACTS

### 1. Validation Report
- **File**: `.agent/artifacts/validation_report.md`
- **Action**: OVERWRITE with the full **Validation Report**.


---

## INPUT PARAMETERS

### Required Inputs
```
{execution_log}     : .agent/artifacts/refactor_execution_log.md (from Phase 5)
{original_context}  : .agent/artifacts/context_summary.md (baseline comparison)
{refactor_plan}     : .agent/artifacts/refactor_plan.md (objective verification)
```

### Optional Inputs
```
{test_scope}        : Test execution scope
                      - "quick"  : Unit tests only (fast feedback)
                      - "full"   : Unit + integration tests (DEFAULT)
                      - "all"    : Unit + integration + UI tests (comprehensive)
```

---

## VALIDATION GATE SYSTEM

This tool implements a **3-tiered quality gate system**:

```
GATE 1: HARD REQUIREMENTS (Must Pass ALL - Non-Negotiable)
  - Zero compiler errors
  - Zero compiler warnings (or no new warnings)
  - Zero data race violations (Swift 6.3 strict concurrency)
  - 100% existing tests passing
  - Behavior preservation verified
  
  FAILURE CONSEQUENCE: ❌ VALIDATION FAILED - STOP immediately, report to user

GATE 2: QUALITY TARGETS (Must Pass 4/5 - Excellence Metrics)
  - Force unwraps reduced 80%+ (remaining justified)
  - Public API documentation 100%
  - Success criteria compliance 85%+
  - Cyclomatic complexity average <10
  - Code coverage maintained or improved
  
  FAILURE CONSEQUENCE: ⚠️ PASSED WITH WARNINGS - Report gaps

GATE 3: CODE REVIEW QUALITY (Subjective Assessment - Guidance)
  - Code readability improved
  - Code testability improved
  - Code maintainability improved
  
  FAILURE CONSEQUENCE: ℹ️ INFORMATIONAL - Note in report
```

---

## EXECUTION PROTOCOL

### PHASE 0: Pre-Validation Checks

Verify prerequisites before starting validation:

```
VERIFY execution_log:
  IF NOT exists at .agent/artifacts/refactor_execution_log.md:
    ABORT: "No execution log found at .agent/artifacts/refactor_execution_log.md. Run /apply-refactor first."
  
  PARSE execution_log as JSON
  EXTRACT changes_completed list
  
  IF changes_completed is empty:
    ABORT: "No changes found in execution log."

VERIFY file accessibility:
  FOR EACH file IN execution_log.files_modified:
    IF NOT file_exists(file):
      ABORT: "Modified file not found: {file}"

CHECK working directory status:
  RUN: git status --porcelain
  
  IF merge conflicts detected:
    ABORT: "Merge conflicts detected. Resolve before validation."
  
  IF untracked build artifacts present:
    WARN: "Untracked files detected - may interfere with build"

LOAD baseline metrics FROM original_context:
  baseline_force_unwraps = context.metrics.force_unwrap_count
  baseline_warnings = context.metrics.compiler_warnings
  baseline_test_count = context.metrics.test_count
  baseline_coverage = context.metrics.code_coverage
  
  STORE for comparison in validation steps
```

### PHASE 1: Build Validation

Execute compilation and concurrency verification:

```
STEP 1.1: Standard Build Compilation
  
  DETERMINE build system:
    IF Package.swift exists:
      RUN: swift build --configuration debug
    ELSE IF *.xcodeproj exists:
      EXTRACT scheme_name FROM project
      RUN: xcodebuild -scheme {scheme_name} -configuration Debug build
  
  CAPTURE build output
  
  PARSE build results:
    COUNT errors
    COUNT warnings
    EXTRACT error messages IF errors > 0
    EXTRACT warning messages IF warnings > 0
  
  EVALUATE compilation status:
    IF errors > 0:
      MARK: gate1_compilation = FAILED
      GOTO: Phase 10 (Failure Reporting)
    
    IF warnings > baseline_warnings:
      new_warnings = warnings - baseline_warnings
      MARK: gate1_warnings = FAILED
      GOTO: Phase 10 (Failure Reporting)
    ELSE IF warnings < baseline_warnings:
      NOTE: "Warnings reduced by {baseline_warnings - warnings}"
      MARK: gate1_warnings = PASSED
    ELSE:
      MARK: gate1_warnings = PASSED

STEP 1.2: Swift 6.3 Strict Concurrency Verification
  
  RUN: swift build -Xswiftc -strict-concurrency=complete
  
  PARSE concurrency warnings:
    SEARCH output FOR:
      - "data race"
      - "Sendable" violations
      - "@MainActor" violations
      - "actor isolation" violations
    
    COUNT concurrency_warnings
  
  EVALUATE concurrency status:
    IF concurrency_warnings > 0:
      MARK: gate1_concurrency = FAILED
      CAPTURE all concurrency warning details
      GOTO: Phase 10 (Failure Reporting)
    ELSE:
      MARK: gate1_concurrency = PASSED

RECORD build metrics:
  build_duration = {seconds}
  modules_compiled = {count}
  swift_files_compiled = {count}
```

### PHASE 2: Test Execution

Execute test suites based on test_scope:

```
STEP 2.1: Unit Test Execution
  
  DETERMINE test command:
    IF Package.swift exists:
      test_command = "swift test"
    ELSE:
      test_command = "xcodebuild test -scheme {scheme} -destination {destination}"
  
  RUN: {test_command}
  
  PARSE test results:
    EXTRACT total_tests
    EXTRACT passed_tests
    EXTRACT failed_tests
    EXTRACT skipped_tests
    EXTRACT execution_time
    
    IF failed_tests > 0:
      FOR EACH failure:
        EXTRACT test_name
        EXTRACT failure_reason
        EXTRACT assertion_details
        EXTRACT stack_trace
        EXTRACT expected_vs_actual
        
        CROSS_REFERENCE with execution_log.changes
        IDENTIFY suspected_change_id
  
  EVALUATE test status:
    IF failed_tests > 0:
      MARK: gate1_tests = FAILED
      GOTO: Phase 10 (Failure Reporting)
    
    IF total_tests < baseline_test_count:
      MARK: gate1_tests = FAILED
      REASON: "Test count decreased"
      GOTO: Phase 10 (Failure Reporting)
    
    ELSE:
      MARK: gate1_tests = PASSED
      
      new_tests_added = total_tests - baseline_test_count
      IF new_tests_added > 0:
        NOTE: "{new_tests_added} new tests added"

STEP 2.2: Integration Test Execution (Conditional)
  
  IF test_scope IN ["full", "all"]:
    RUN: swift test --filter IntegrationTests
    
    PARSE results (same criteria as unit tests)
    
    IF any failures:
      MARK: gate1_integration = FAILED
      GOTO: Phase 10 (Failure Reporting)
  ELSE:
    MARK: integration_tests = SKIPPED
    NOTE: "Run with test_scope='full' for integration tests"

STEP 2.3: UI Test Execution (Conditional)
  
  IF test_scope == "all":
    RUN: xcodebuild test -scheme {scheme} -only-testing:{UITestTarget}
    
    PARSE results (same criteria)
    
    IF any failures:
      MARK: gate1_ui_tests = FAILED
      GOTO: Phase 10 (Failure Reporting)
  ELSE:
    MARK: ui_tests = SKIPPED

STEP 2.4: Code Coverage Analysis (If Available)
  
  IF coverage tools available:
    RUN: swift test --enable-code-coverage
    RUN: xcrun llvm-cov report {paths}
    
    PARSE coverage report:
      EXTRACT overall_coverage_percentage
      EXTRACT module_coverage_breakdown
    
    COMPARE with baseline:
      coverage_change = overall_coverage - baseline_coverage
      
      IF overall_coverage < baseline_coverage:
        MARK: gate2_coverage = WARNING
        NOTE: "Coverage decreased by {abs(coverage_change)}%"
      ELSE:
        MARK: gate2_coverage = PASSED
        NOTE: "Coverage: {overall_coverage}% ({coverage_change:+}%)"
```

### PHASE 3: Code Quality Metrics

Calculate and evaluate quality metrics:

```
STEP 3.1: Force Unwrap Analysis
  
  FOR EACH file IN execution_log.files_modified:
    SCAN file FOR patterns:
      - "!" (force unwrap operator)
      - "try!" (force try)
      - "as!" (force cast)
    
    COUNT occurrences
    EXTRACT line numbers
    CHECK for justification comments (within 1 line)
    
    ADD to force_unwrap_results
  
  CALCULATE metrics:
    current_unwraps = SUM(force_unwrap_results)
    reduction_count = baseline_force_unwraps - current_unwraps
    reduction_percentage = (reduction_count / baseline_force_unwraps) × 100
  
  EVALUATE against gate:
    IF reduction_percentage >= 80:
      MARK: gate2_unwraps = PASSED
    ELSE:
      MARK: gate2_unwraps = FAILED
    
    FOR EACH remaining unwrap WITHOUT justification:
      FLAG as unjustified

STEP 3.2: Cyclomatic Complexity Analysis
  
  FOR EACH modified function IN execution_log:
    CALCULATE complexity:
      complexity = 1  // base complexity
      
      COUNT decision points:
        - if statements
        - guard statements
        - for loops
        - while loops
        - case statements in switch
        - && operators
        - || operators
        - ?? operators
      
      complexity += decision_point_count
    
    RECORD function_complexity
    
    IF complexity > 15:
      FLAG as "EXCEEDS HARD LIMIT"
    ELSE IF complexity > 10:
      FLAG as "ABOVE TARGET"
  
  CALCULATE average:
    average_complexity = SUM(complexities) / COUNT(functions)
  
  EVALUATE against gate:
    IF average_complexity < 10 AND no_functions_above_15:
      MARK: gate2_complexity = PASSED
    ELSE:
      MARK: gate2_complexity = FAILED

STEP 3.3: Function Length Analysis
  
  FOR EACH modified function:
    COUNT lines_of_code:
      EXCLUDE blank lines
      EXCLUDE comment-only lines
      INCLUDE actual code lines
    
    RECORD function_length
    
    IF length > 80:
      FLAG as "EXCEEDS HARD LIMIT"
    ELSE IF length > 50:
      FLAG as "ABOVE TARGET"
  
  CALCULATE average:
    average_length = SUM(lengths) / COUNT(functions)
  
  EVALUATE (informational only - not gate)
```

### PHASE 4: Compliance Verification

Verify adherence to governance and standards:

```
STEP 4.1: Success Criteria Compliance
  
  LOAD success criteria definitions from .agent/core/success-criteria.md
  
  FOR EACH modified file:
    FOR EACH criterion IN [1..16]:
      APPLY criterion rules to file
      
      COUNT total_applicable_items
      COUNT passed_items
      
      IF total_applicable_items > 0:
        criterion_pass_rate = (passed_items / total_applicable_items) × 100
        
        CLASSIFY:
          IF criterion_pass_rate >= 85: PASSED
          ELSE IF criterion_pass_rate >= 70: WARNING
          ELSE: FAILED
      ELSE:
        MARK criterion as N/A
  
  CALCULATE overall_compliance:
    total_passed = SUM(all passed_items)
    total_applicable = SUM(all applicable_items)
    overall_percentage = (total_passed / total_applicable) × 100
  
  EVALUATE against gate:
    IF overall_percentage >= 85:
      MARK: gate2_compliance = PASSED
    ELSE:
      MARK: gate2_compliance = FAILED

STEP 4.2: Governance Document Compliance
  
  IF ENGINEERING_STANDARDS.md exists:
    LOAD standards document
    
    VERIFY modified code follows:
      - Naming conventions (check patterns)
      - Code organization (check MARK sections)
      - Architecture requirements (check patterns)
      - Dependency patterns (check imports)
    
    MARK: standards_compliant = TRUE/FALSE

  IF CODE_TEMPLATES.md exists:
    VERIFY new patterns match templates
    MARK: templates_compliant = TRUE/FALSE
```

### PHASE 5: Concurrency Safety Audit

Deep verification of Swift 6.3 concurrency safety:

```
STEP 5.1: Non-Sendable Types Crossing Boundaries
  
  SCAN all modified files FOR patterns:
    
    PATTERN 1: Task with non-Sendable capture
      Task {
        await function(non_sendable_variable)
      }
    
    PATTERN 2: Actor methods with non-Sendable parameters
      actor MyActor {
        func process(_ item: NonSendableType)
      }
    
    PATTERN 3: Async functions with non-Sendable parameters
      func asyncProcess(_ data: NonSendableType) async
  
  FOR EACH detected pattern:
    VERIFY type conformance:
      IF type DOES NOT conform to Sendable:
        FLAG violation
        RECORD location
  
  IF violations found:
    MARK: concurrency_safety = FAILED
    NOTE: "Non-Sendable types crossing boundaries"

STEP 5.2: @MainActor Coverage Verification
  
  SCAN for UI-touching code:
    - Classes inheriting from ObservableObject
    - Classes inheriting from UIViewController
    - Classes inheriting from UIView
    - SwiftUI View body modifications
  
  FOR EACH UI-touching class:
    CHECK for @MainActor annotation
    
    IF missing @MainActor:
      FLAG violation
      RECORD class_name and location
  
  IF violations found:
    MARK: main_actor_coverage = FAILED

STEP 5.3: Actor Isolation Verification
  
  SCAN for actor usage:
    FOR EACH actor declaration:
      FIND all property/method access points
      
      FOR EACH access:
        IF NOT preceded by await:
          FLAG isolation violation
  
  IF violations found:
    MARK: actor_isolation = FAILED

STEP 5.4: Global Mutable State Detection
  
  SCAN for global state:
    PATTERN: static var {name}: {mutable_type}
    
    FOR EACH detected:
      CHECK if protected by:
        - actor isolation
        - @MainActor
        - locks/synchronization
      
      IF no protection:
        FLAG as unsafe global state
  
  IF unsafe globals found:
    MARK: global_state_safety = FAILED
```

### PHASE 6: Objective Achievement Verification

Verify original objectives from Phase 1 were met:

```
LOAD original objectives FROM original_context

FOR EACH objective:
  CROSS_REFERENCE with execution_log.changes
  
  DETERMINE status:
    IF objective fully addressed by changes:
      MARK: ✅ COMPLETE
    ELSE IF objective partially addressed:
      MARK: ⚠️ PARTIAL
      CALCULATE percentage_complete
    ELSE:
      MARK: ❌ NOT MET
  
  GATHER evidence:
    - Which changes addressed this objective
    - Test results proving it works
    - Metrics showing improvement

CALCULATE overall achievement:
  achievement_score = (complete_count + 0.5 × partial_count) / total_objectives

VERIFY behavioral preservation:
  EVIDENCE: All existing tests passed without modification
  
  IF all_existing_tests_passed:
    MARK: behavior_preserved = TRUE
  ELSE:
    MARK: behavior_preserved = FALSE
    FLAG: "Behavioral changes detected"
```

### PHASE 7: Documentation Audit

Verify documentation completeness:

```
STEP 7.1: Public API Documentation Check
  
  FOR EACH file IN execution_log.files_modified:
    SCAN for public/open declarations:
      - public class/struct/enum/actor
      - public func/var/let
      - open class/func/var
    
    FOR EACH public declaration:
      CHECK for DocC comment (///)
      
      IF missing documentation:
        FLAG violation
      ELSE:
        VERIFY documentation includes:
          - Summary line
          - Parameters (if applicable)
          - Returns (if applicable)
          - Throws (if applicable)
        
        IF incomplete:
          FLAG incomplete_documentation
  
  CALCULATE coverage:
    doc_coverage = documented_apis / total_public_apis
  
  EVALUATE against gate:
    IF doc_coverage == 1.0:
      MARK: gate2_documentation = PASSED
    ELSE:
      MARK: gate2_documentation = FAILED

STEP 7.2: Inline Comment Verification
  
  CHECK for required comments:
    - @unchecked Sendable justifications
    - Force unwrap safety comments
    - Complex logic explanations
    - MARK sections for organization
  
  FLAG any missing required comments
```

### PHASE 8: Gate Evaluation

Evaluate all collected metrics against gates:

```
EVALUATE GATE 1 (Must Pass ALL):
  
  gate1_results = {
    "compiler_errors": errors == 0,
    "compiler_warnings": warnings <= baseline_warnings,
    "data_races": concurrency_warnings == 0,
    "tests_passing": failed_tests == 0,
    "behavior_preserved": all_existing_tests_passed
  }
  
  gate1_passed = ALL(gate1_results.values() == TRUE)
  
  IF NOT gate1_passed:
    validation_status = "FAILED"
    GOTO: Phase 10 (Failure Reporting)

EVALUATE GATE 2 (Must Pass 4/5):
  
  gate2_results = {
    "force_unwrap_reduction": reduction_percentage >= 80,
    "public_api_docs": doc_coverage == 1.0,
    "compliance": overall_percentage >= 85,
    "complexity": average_complexity < 10,
    "coverage": overall_coverage >= baseline_coverage
  }
  
  gate2_passed_count = COUNT(gate2_results.values() == TRUE)
  
  IF gate2_passed_count >= 4:
    gate2_status = "PASSED"
  ELSE:
    gate2_status = "WARNING"

EVALUATE GATE 3 (Subjective):
  
  ASSESS readability (based on):
    - Async/await vs callbacks
    - Naming clarity
    - Function decomposition
  
  ASSESS testability (based on):
    - Dependency injection usage
    - Protocol abstractions
    - Reduced coupling
  
  ASSESS maintainability (based on):
    - Complexity reduction
    - Documentation improvement
    - Error handling clarity

DETERMINE final validation status:
  IF gate1_passed AND gate2_passed_count >= 4:
    validation_status = "PASSED"
  ELSE IF gate1_passed AND gate2_passed_count >= 3:
    validation_status = "PASSED_WITH_WARNINGS"
  ELSE:
    validation_status = "FAILED"
```

### PHASE 9: Report Generation (Success Path)

Generate comprehensive validation report:

```
IF validation_status IN ["PASSED", "PASSED_WITH_WARNINGS"]:
  
  GENERATE report using output template (see OUTPUT SPECIFICATION)
  
  INCLUDE all sections:
    - Executive Summary
    - Gate 1 Results (detailed table)
    - Gate 2 Results (detailed table)
    - Build Validation (compilation + concurrency)
    - Test Execution (all test types)
    - Code Quality Metrics (unwraps, complexity, length)
    - Compliance Checks (criteria + governance)
    - Concurrency Safety Audit (4 subchecks)
    - Objective Achievement (original goals)
    - Documentation Review (public APIs + inline)
    - Final Quality Gates Summary
    - Validation Verdict (ready for Phase 7?)
    - Recommendations for Future
  
  SET ready_for_phase_7 = TRUE
  
  RETURN report
```

### PHASE 10: Failure Handling & Rollback Options

Handle validation failures with actionable guidance:

```
IF validation_status == "FAILED":
  
  IDENTIFY failure category:
    IF gate1_compilation FAILED:
      failure_type = "COMPILATION_ERROR"
    ELSE IF gate1_tests FAILED:
      failure_type = "TEST_FAILURE"
    ELSE IF gate1_concurrency FAILED:
      failure_type = "CONCURRENCY_VIOLATION"
    ELSE IF gate1_warnings FAILED:
      failure_type = "NEW_WARNINGS"
  
  GENERATE failure report:
    
    INCLUDE:
      - Which gate failed
      - Exact error/failure details
      - File and line numbers
      - Suspected root cause
      - Cross-reference with change IDs from execution_log
    
    PROVIDE rollback options:
      ```
      ROLLBACK OPTIONS:
      
      1. Rollback ALL changes
         Command: git reset --hard {pre_refactor_commit}
         Impact: Complete revert to pre-refactor state
         Risk: Low - guaranteed working state
      
      2. Rollback specific change: {suspected_change_id}
         Command: git revert {commit_hash_for_change}
         Impact: Removes only problematic change
         Risk: Medium - may have dependencies
      
      3. Attempt manual fix
         Approach: {suggested_fix_based_on_error}
         Impact: Preserve most changes, fix issue
         Risk: High - requires investigation
      
      4. Request human review
         Action: Analyze error details with developer
         Impact: Expert guidance on resolution
         Risk: Low - informed decision
      ```
  
  STOP execution
  
  WAIT for user decision
  
  DO NOT proceed to Phase 7
  DO NOT attempt automatic fixes
```

---

## OPERATIONAL CONSTRAINTS

### Forbidden Actions
```
❌ NEVER auto-fix validation failures (always report and wait)
❌ NEVER suppress test failures (every failure MUST be addressed)
❌ NEVER lower quality standards to force passage
❌ NEVER proceed to Phase 7 if Gate 1 fails
❌ NEVER dismiss compiler warnings as "minor"
❌ NEVER skip concurrency safety checks
❌ NEVER accept decreased test coverage without justification
❌ NEVER modify code during validation (read-only phase)
```

### Required Actions
```
✅ ALWAYS run full test suite (unless test_scope explicitly limits)
✅ ALWAYS verify build with strict concurrency enabled
✅ ALWAYS check against governance documents
✅ ALWAYS calculate exact compliance scores (not estimates)
✅ ALWAYS verify Sendable conformance at boundaries
✅ ALWAYS compare against original objectives
✅ ALWAYS generate detailed report (success or failure)
✅ ALWAYS provide rollback options on failure
```

---

## SWIFT 6.3 VERIFICATION CHECKLIST

Critical Swift 6.3-specific validations:

```
SENDABLE COMPLIANCE:
  [ ] All types passed to Task {} are Sendable
  [ ] All actor method parameters are Sendable (or actor-isolated)
  [ ] All async function parameters crossing isolation are Sendable
  [ ] No @unchecked Sendable without inline justification comment

MAIN ACTOR COMPLIANCE:
  [ ] All SwiftUI @Observable classes have @MainActor
  [ ] All ObservableObject classes have @MainActor
  [ ] All UIKit ViewControllers have @MainActor (if strict concurrency)
  [ ] SwiftUI Views have implicit/explicit @MainActor

ACTOR ISOLATION:
  [ ] Shared mutable state wrapped in actors
  [ ] No unprotected global static var
  [ ] Actor methods called with await
  [ ] No isolation violations detected in build

STRUCTURED CONCURRENCY:
  [ ] Task used for unstructured concurrency only when necessary
  [ ] async let or withTaskGroup for parallel tasks
  [ ] Task cancellation checked in long-running loops
  [ ] No Task.detached without explicit justification
```

---

## OUTPUT SPECIFICATION

Generate structured Markdown validation report:

### Report Template

```markdown
# Validation Report - Swift 6.3 Refactoring

**Project**: {ProjectName}  
**Validated**: {ISO 8601 timestamp}  
**Validation Scope**: {test_scope}  
**Overall Status**: {✅ PASSED | ⚠️ PASSED WITH WARNINGS | ❌ FAILED}

---

## Executive Summary

**Final Verdict**: {status_emoji} **{STATUS}** - {one-line summary}

**Changes Validated**: {N} changes across {M} files  
**Success Rate**: {percentage}% ({passed}/{total} changes passed)

**Key Achievements**:
- ✅ {achievement 1 with metrics}
- ✅ {achievement 2 with metrics}
- ✅ {achievement 3 with metrics}
- {additional achievements}

**Quality Score**: {overall_percentage}% ({EXCEEDS | MEETS | BELOW} 85% target)

---

## Gate 1: Hard Requirements {✅ PASSED | ❌ FAILED}

| Requirement | Status | Details |
|-------------|--------|---------|
| Zero compiler errors | {✅ PASS | ❌ FAIL} | {count} errors |
| Zero compiler warnings | {✅ PASS | ❌ FAIL} | {count} warnings (baseline: {baseline}) |
| Zero data race violations | {✅ PASS | ❌ FAIL} | Swift 6.3 strict concurrency: {status} |
| 100% tests passing | {✅ PASS | ❌ FAIL} | {passed}/{total} tests passed |
| Behavior preserved | {✅ PASS | ❌ FAIL} | {status description} |

**Result**: {✅ ALL HARD REQUIREMENTS MET | ❌ GATE 1 FAILED - VALIDATION STOPPED}

{IF FAILED: Include detailed failure information and rollback options}

---

## Gate 2: Quality Targets {✅ N/5 PASSED}

| Target | Status | Metric |
|--------|--------|--------|
| Force unwrap reduction 80%+ | {✅ PASS | ❌ FAIL} | {percentage}% reduction ({before} → {after}) |
| Public API docs 100% | {✅ PASS | ❌ FAIL} | {documented}/{total} APIs documented |
| Checklist compliance 85%+ | {✅ PASS | ❌ FAIL} | {percentage}% ({passed}/{applicable} criteria) |
| Cyclomatic complexity <10 | {✅ PASS | ❌ FAIL} | Average: {avg_complexity} |
| Code coverage maintained | {✅ PASS | ❌ FAIL} | {before}% → {after}% ({change:+}) |

**Result**: {✅ ALL QUALITY TARGETS MET | ⚠️ {N}/5 TARGETS MET}

---

## Build Validation {✅ SUCCESS | ❌ FAILED}

### Compilation Status

```
{✅ | ❌} Build: {STATUS}
   Swift version: 6.3
   Configuration: Debug
   Duration: {seconds}s
   
   Errors: {count}
   Warnings: {count}
   
   Modules compiled: {count}
   Swift files: {count}
```

{IF errors or warnings: List each with file:line and message}

### Swift 6.3 Strict Concurrency Check

```
{✅ | ❌} Strict Concurrency: {STATUS}
   
   Command: swift build -Xswiftc -strict-concurrency=complete
   
   Data race warnings: {count}
   Sendable violations: {count}
   @MainActor violations: {count}
   Actor isolation violations: {count}
   
   Status: {CLEAN | VIOLATIONS DETECTED}
```

**Baseline Comparison**:
- Before refactor: {count} warnings
- After refactor: {count} warnings
- **{Improvement | Regression}**: {change description}

---

## Test Execution {✅ ALL PASSED | ❌ FAILURES}

### Unit Tests

```
{✅ | ❌} Unit Tests: {passed}/{total} PASSED ({percentage}%)
   
   Duration: {seconds}s
   
   Test Suites: {count}
   Total Tests: {count}
   Passed: {count} ✅
   Failed: {count}
   Skipped: {count}
   
   New Tests Added: {count}
   Tests Modified: {count}
```

**Test Breakdown by Suite**:
{FOR EACH suite: - {SuiteName}: {passed}/{total} {✅ | ❌}}

{IF failures: Include detailed failure information}

### Integration Tests

```
{✅ | ⚠️ | ❌} Integration Tests: {STATUS}
   
   {IF SKIPPED: Note about test_scope parameter}
   {ELSE: Similar metrics as unit tests}
```

### UI Tests

```
{✅ | ⚠️ | ❌} UI Tests: {STATUS}
   
   {Similar format}
```

### Coverage Report

```
Code Coverage: {percentage}% ({change:+}% from baseline)
   
   Before refactor: {baseline}%
   After refactor: {current}%
   
   Coverage by module:
   {FOR EACH module: - {ModuleName}: {percentage}% {✅ | ⚠️}}
```

---

## Code Quality Metrics {✅ EXCELLENT | ⚠️ ACCEPTABLE | ❌ NEEDS IMPROVEMENT}

### Force Unwraps Analysis

**Baseline**: {count} force unwraps across {files} files  
**Current**: {count} force unwraps  
**Reduction**: {percentage}% {✅ | ❌} (target: 80%+)

**Remaining Force Unwraps** ({IF any: all justified | NONE}):

{FOR EACH remaining unwrap:}
{number}. `{File.swift}:{line}`
   ```swift
   {code snippet with unwrap}
   ```
   **Justification**: {comment or "⚠️ MISSING JUSTIFICATION"}

**Result**: {✅ PASSED | ❌ FAILED}

### Cyclomatic Complexity

**Analysis**: {count} functions modified

| Complexity Range | Count | Status |
|------------------|-------|--------|
| 1-5 (Simple) | {count} | ✅ |
| 6-10 (Moderate) | {count} | ✅ |
| 11-15 (Complex) | {count} | ⚠️ |
| 16+ (Very Complex) | {count} | ❌ |

**Average Complexity**: {average} {✅ | ❌} (target: <10)  
**Maximum Complexity**: {max} ({function_name} in {file})

**Flagged Functions**:
{FOR EACH flagged function:}
- `{ClassName}.{methodName}()`: Complexity {value}
  - **Recommendation**: {suggestion}
  - **Status**: {Acceptable | Needs Refactoring}

**Result**: {✅ PASSED | ❌ FAILED}

### Function Length

**Analysis**: {count} functions modified

| Line Range | Count | Status |
|------------|-------|--------|
| 1-30 lines | {count} | ✅ |
| 31-50 lines | {count} | ✅ |
| 51-80 lines | {count} | ⚠️ |
| 81+ lines | {count} | ❌ |

**Average Length**: {lines} lines {✅ | ⚠️} (target: <50)

**Flagged Functions**:
{FOR EACH > 50 lines:}
- `{ClassName}.{methodName}()`: {lines} lines
  - **Assessment**: {cohesive/acceptable or needs split}
  - **Status**: {✅ | ⚠️ | ❌}

**Result**: {✅ PASSED | ⚠️ ACCEPTABLE | ❌ FAILED}

---

## Compliance Checks {✅ COMPLIANT | ⚠️ PARTIAL | ❌ NON-COMPLIANT}

### Success Criteria Checklist

**Overall Compliance**: {percentage}% ({passed}/{applicable} criteria) {✅ | ❌} (target: 85%+)

| # | Criterion | Status | Pass Rate |
|---|-----------|--------|-----------|
| 1 | Strict Concurrency Compliance | {✅ | ⚠️ | ❌ | ➖} | {percentage}% ({passed}/{total}) |
| 2 | Modern Async/Await Patterns | {status} | {percentage}% |
| ... | ... | ... | ... |
| 16 | Documentation & Comments | {status} | {percentage}% |

**Items Not Passed**:
{FOR EACH failed item:}
- Criterion {#}, Item {#}: {description}

**Result**: {✅ PASSED | ❌ FAILED}

### ENGINEERING_STANDARDS.md Compliance

```
{✅ | ⚠️ | ❌} {STATUS}

- Naming conventions: {✅ | ❌} {details}
- Code organization: {✅ | ❌} {details}
- Architecture patterns: {✅ | ❌} {details}
- Dependency patterns: {✅ | ❌} {details}
```

### CODE_TEMPLATES.md Compliance

```
{✅ | ⚠️ | ❌} {STATUS}

- {Template type}: {✅ | ❌} {details}
{FOR EACH template checked}
```

---

## Sendable & Concurrency Safety Audit {✅ VERIFIED | ❌ VIOLATIONS}

### Non-Sendable Types Check

**Scan Result**: {✅ All types crossing boundaries are Sendable | ❌ Violations detected}

**Types Verified**:
{FOR EACH type crossing boundaries:}
- `{TypeName}`: {✅ Sendable | ❌ Non-Sendable | ✅ @unchecked Sendable} {details}

**{No violations detected | Violations found}** {✅ | ❌}

{IF violations: List each with location and recommendation}

### @MainActor Coverage

**Scan Result**: {✅ All UI code has @MainActor | ❌ Missing annotations}

**Classes Verified**:
{FOR EACH UI class:}
- `{ClassName}`: {✅ @MainActor applied | ❌ Missing @MainActor}

**{No violations detected | Violations found}** {✅ | ❌}

### Actor Isolation

**Scan Result**: {✅ No isolation violations | ❌ Violations detected}

**Actors in Project**:
{FOR EACH actor:}
- `{ActorName}`: {✅ Properly isolated | ❌ Violations found}

**{No violations detected | Violations found}** {✅ | ❌}

### Global Mutable State

**Scan Result**: {✅ No unprotected global state | ❌ Unsafe globals found}

**All global state reviewed**:
{FOR EACH global:}
- `{ClassName}.{property}`: {✅ | ❌} {protection mechanism}

**{No violations detected | Violations found}** {✅ | ❌}

---

## Regression Check {✅ ALL OBJECTIVES MET | ⚠️ PARTIAL}

### Original Objectives (from Phase 1 Context Summary)

{FOR EACH objective from original_context:}
{number}. {✅ | ⚠️ | ❌} **{objective description}**
   - Status: {COMPLETE | PARTIAL | NOT MET}
   - Evidence: {specific changes and test results}
   {IF PARTIAL: - Progress: {percentage}% complete}

**Overall**: {complete_count}/{total} objectives met ({percentage}%)

### Behavioral Preservation

**Verification Method**: {description of how behavior was verified}

- {✅ | ❌} No unintended behavioral changes detected
- {✅ | ❌} All original functionality verified working
- {✅ | ❌} Public API surface unchanged (or deprecated properly)
- {✅ | ❌} Backward compatibility maintained

**Result**: {✅ BEHAVIOR PRESERVED | ❌ BEHAVIORAL CHANGES}

---

## Documentation Review {✅ COMPLETE | ⚠️ PARTIAL | ❌ INCOMPLETE}

### Public API Documentation

**Status**: {✅ | ❌} {percentage}% coverage ({documented}/{total} APIs)

**Documented APIs**:
{FOR EACH public API:}
{number}. `{signature}` - {✅ | ❌} {completeness notes}

{IF any missing: List undocumented APIs}

**Result**: {✅ PASSED | ❌ FAILED}

### Inline Comments

**Review Result**: {✅ Adequate | ⚠️ Partial | ❌ Insufficient}

- {✅ | ❌} Complex logic has explanatory comments
- {✅ | ❌} @unchecked Sendable has justification ({count} instances)
- {✅ | ❌} Force unwraps have safety comments ({count} instances)
- {✅ | ❌} MARK sections organize code ({count} files)

### Change Documentation

**Execution Log Review**: {✅ Complete | ⚠️ Partial | ❌ Missing}

- {✅ | ❌} Rationale provided for each change
- {✅ | ❌} Success criteria referenced
- {✅ | ❌} Risk assessment documented

---

## Final Quality Gates Summary

### Gate 1: Hard Requirements {✅ | ❌} {N}/5 PASSED

| Requirement | Result |
|-------------|--------|
| Zero compiler errors | {✅ PASS | ❌ FAIL} |
| Zero compiler warnings | {✅ PASS | ❌ FAIL} |
| Zero data race violations | {✅ PASS | ❌ FAIL} |
| 100% tests passing | {✅ PASS | ❌ FAIL} |
| Behavior preserved | {✅ PASS | ❌ FAIL} |

### Gate 2: Quality Targets {✅ | ⚠️} {N}/5 PASSED

| Target | Result |
|--------|--------|
| Force unwrap reduction 80%+ | {✅ PASS | ❌ FAIL} ({percentage}%) |
| Public API docs 100% | {✅ PASS | ❌ FAIL} ({percentage}%) |
| Checklist compliance 85%+ | {✅ PASS | ❌ FAIL} ({percentage}%) |
| Cyclomatic complexity <10 | {✅ PASS | ❌ FAIL} (avg {value}) |
| Code coverage maintained | {✅ PASS | ❌ FAIL} ({before}% → {after}%) |

### Gate 3: Code Review Quality {✅ EXCELLENT | ⚠️ GOOD | ❌ POOR}

- {✅ | ⚠️ | ❌} Code is **more readable** than before
  - Evidence: {specific improvements}
- {✅ | ⚠️ | ❌} Code is **more testable** than before
  - Evidence: {specific improvements}
- {✅ | ⚠️ | ❌} Code is **more maintainable** than before
  - Evidence: {specific improvements}

---

## Validation Verdict

**STATUS**: {✅ VALIDATION PASSED | ⚠️ VALIDATION PASSED WITH WARNINGS | ❌ VALIDATION FAILED}

**Ready for Phase 7**: {YES | NO} - {explanation}

**Confidence Level**: {HIGH | MEDIUM | LOW}
- {assessment of overall quality}
- {blocking issues if any}
- {recommendations if any}

## Next Steps

{IF Ready for Phase 7 == YES:}
👉 **Run `/explain-changes`** to generate the final documentation and rationale.
{ELSE:}
🛑 **Resolve failures** before proceeding.
{END IF}

---

## Recommendations for Future Refactoring

{IF any improvement opportunities:}
{FOR EACH recommendation:}
{number}. {⚠️ | ℹ️} **{recommendation title}**
   - {detailed suggestion}
   - {priority level}
   - {blocking status}

{IF no recommendations: Note that code quality is excellent}

---

*Validation complete. {Final summary statement.}*
```

### Output Validation Requirements

```
VERIFY report contains:
  ✅ All gates evaluated with explicit pass/fail
  ✅ All metrics calculated (not estimated)
  ✅ All test results with exact counts
  ✅ Baseline comparisons for all applicable metrics
  ✅ Specific file:line references for violations
  ✅ Rollback options if failed
  ✅ Clear verdict on Phase 7 readiness

FORMATTING:
  - Use proper Markdown syntax
  - Code blocks have language identifier
  - Tables properly formatted
  - Status emojis consistent (✅⚠️❌➖)
  - Sections separated with ---
```

---

## USAGE EXAMPLES

### Example 1: Full Validation (Success)
```
User: /validate-changes test_scope="full"
Agent: [Executes complete validation suite]
Agent: ✅ VALIDATION PASSED - All quality gates met
       
       Gate 1: 5/5 PASSED ✅
       Gate 2: 5/5 PASSED ✅
       
       Highlights:
       - 47/47 tests passed
       - Zero compiler errors/warnings
       - Zero data race violations
       - 92% force unwrap reduction
       - 92% compliance score
       
       Ready for Phase 7 (Documentation)
```

### Example 2: Quick Validation (Warnings)
```
User: /validate-changes test_scope="quick"
Agent: [Executes quick validation]
Agent: ⚠️ VALIDATION PASSED WITH WARNINGS
       
       Gate 1: 5/5 PASSED ✅
       Gate 2: 3/5 PASSED ⚠️
       
       Warnings:
       - Public API docs: 75% (target: 100%)
       - Code coverage: 78% → 76% (-2%)
       
       Ready for Phase 7, but recommend addressing warnings
```

### Example 3: Validation Failure
```
User: /validate-changes
Agent: ❌ VALIDATION FAILED - Test failures detected
       
       GATE 1 FAILURE: Tests
       - 2/47 tests failed (BLOCKING)
       
       Failed Tests:
       1. UserViewModelTests.testFetchUser
          Error: Expected User, got nil
          Location: Tests/UserViewModelTests.swift:45
          Suspected change: CHANGE-003 (async conversion)
          
       2. NetworkServiceTests.testErrorHandling
          Error: Unexpected error type NetworkError
          Location: Tests/NetworkServiceTests.swift:89
          Suspected change: CHANGE-004 (error enum refactor)
       
       ROLLBACK OPTIONS:
       
       1. Rollback all changes
          Command: git reset --hard abc1234
          Risk: Low - returns to known good state
       
       2. Rollback CHANGE-003 and CHANGE-004
          Commands: git revert def5678 ghi9012
          Risk: Medium - may have dependencies
       
       3. Investigate and fix manually
          Recommended: Review async conversion logic
          Risk: High - requires debugging
       
       VALIDATION STOPPED - Awaiting user decision
```

---

## CRITICAL OPERATIONAL PRINCIPLES

1. **Gate 1 is Non-Negotiable**: ALL requirements must pass without exception
2. **No Auto-Fixing**: Always report failures and wait for user guidance
3. **Evidence-Based**: Every claim must be backed by metrics or test results
4. **Exact Metrics**: Calculate precise numbers, never estimate or approximate
5. **Swift 6.3 Priority**: Concurrency safety is critical - zero tolerance
6. **Behavioral Preservation**: Tests are the proof - all must pass
7. **Transparent Reporting**: Report both successes and failures clearly

---

*This tool performs comprehensive quality gate validation. All Gate 1 requirements must pass before proceeding to Phase 7. Never skip validation steps or lower standards.*

*Last Updated: 2025-11-26*
