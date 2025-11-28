# review-code

**Workflow Phase**: Phase 3 (Code Quality Assessment & Review)  
**Target**: Swift 6.3 Read-Only Code Review
**Input**: `.agent/artifacts/analysis_report.md` (from Phase 2)

---

## AGENT DIRECTIVE

Execute comprehensive read-only code quality assessment against 16 Success Criteria categories. **You must ingest the Logic & Dependency Report from Phase 2** to prioritize your review. Identify violations, warnings, and improvement opportunities with actionable feedback.

**CRITICAL**: This is a READ-ONLY analysis tool—ZERO code modifications permitted.

**Output**: Structured review report with prioritized findings and compliance scoring.

---

## INPUT PARAMETERS

### Required Inputs
```
1. Analysis Report Artifact:
   Path: .agent/artifacts/analysis_report.md
   Content: Concurrency risks, logic issues, impact analysis from Phase 2.

2. Target Code:
   Access: Read files identified in the Analysis Report.

3. Success Criteria Definitions:
   Path: .agent/core/success-criteria.md
   Content: Detailed rules for the 16 quality categories.
```

### Optional Inputs
```
{checklist_scope}   : Comma-separated criteria IDs (1-16) or "all" (default: "all")
                      Examples: "1,2,3" (concurrency only), "all" (comprehensive)
```

---

## SUCCESS CRITERIA REFERENCE

Apply these 16 quality categories systematically:

```
1.  Strict Concurrency Compliance      (Sendable, @MainActor, actors, data races)
2.  Modern Async/Await Patterns        (async/await, TaskGroup, AsyncSequence)
3.  Safe Optional Handling             (minimize force unwraps, guard let)
4.  Idiomatic Naming Conventions       (camelCase, PascalCase, boolean assertions)
5.  Value Types by Default             (struct over class)
6.  Functions Are Focused & Readable   (SRP, cyclomatic complexity <10, <50 LOC)
7.  Composable Error Handling          (custom Error types, Result, async throws)
8.  Memory Management & Retain Cycles  (weak/unowned, capture lists)
9.  Dependency Injection               (protocol-based, no singletons)
10. Modern Testing Practices           (Swift Testing framework preferred)
11. Type Safety (some/any)             (avoid Any/AnyObject)
12. Swift Macro Usage                  (@Observable, @Model, #Preview)
13. Codable for Serialization          (CodingKeys, custom encode/decode)
14. Security Best Practices            (no hardcoded secrets, Keychain)
15. SwiftUI View Best Practices        (@Observable for iOS 17+, composition)
16. Documentation & Comments           (DocC, MARK sections)
```

---

## EXECUTION PROTOCOL

### PHASE 1: Ingest Analysis Report

```
ACTION: READ .agent/artifacts/analysis_report.md

EXTRACT:
  - High Risk Concurrency Issues (Criterion 1)
  - Complexity Hotspots (Criterion 6)
  - Safety Violations (Criteria 3, 7, 8)
  - Modernization Opportunities (Criterion 2)

PRIORITIZE:
  - If Analysis Report flags "Critical Concurrency Issues", set Criterion 1 to CRITICAL priority.
  - If "Complexity Hotspots" found, set Criterion 6 to HIGH priority.
```

### PHASE 2: Criteria Loading & Scope Definition

```
IF {checklist_scope} == "all":
  SET active_criteria = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16]
ELSE:
  PARSE {checklist_scope} as comma-separated integers
  SET active_criteria = parsed_list
  VALIDATE: All IDs are in range 1-16

FOR EACH criterion_id IN active_criteria:
  LOAD criterion definition from .agent/core/success-criteria.md
  PREPARE evaluation checklist items
```

### PHASE 3: Line-by-Line Code Scanning

Execute systematic code analysis, validating Phase 2 findings and discovering new issues:

```
FOR EACH criterion IN active_criteria:
  FOR EACH line IN {target_code}:
    EVALUATE line against criterion rules
    
    CHECK against Phase 2 Report:
      - Does this confirm a reported risk? (Mark as CONFIRMED)
      - Is this a new finding? (Mark as NEW)
    
    CLASSIFY result:
      ✅ PASS      : Code meets criterion standard
      ⚠️ WARNING   : Minor issue or improvement opportunity
      ❌ VIOLATION : Clear violation of best practice
      ➖ N/A       : Criterion not applicable to this code
    
    IF result is WARNING OR VIOLATION:
      CREATE finding:
        - location (file:line)
        - category (criterion ID)
        - severity (CRITICAL/HIGH/MEDIUM/LOW)
        - current_code (problematic snippet)
        - problem (clear explanation)
        - recommendation (specific fix)
        - rationale (why it matters for Swift 6.3)
```

#### Severity Classification Logic

```
ASSIGN severity based on impact:

CRITICAL:
  - Data race safety violations (Phase 2 confirmed)
  - Non-Sendable types crossing actor boundaries
  - Missing @MainActor on UI code
  - @unchecked Sendable without justification
  - Crashes (force unwraps on network/user/file data)
  - Security vulnerabilities (hardcoded credentials, SQL injection)

HIGH:
  - Force unwraps on potentially nil data
  - Completion handlers (should be async/await)
  - Missing error handling (try!, bare try?)
  - Retain cycle risks in closures
  - Public API missing documentation

MEDIUM:
  - Naming convention violations
  - Missing documentation on internal APIs
  - Functions >50 lines or complexity >10
  - `Any` types (should use generics/protocols)
  - Unnecessary class (should be struct)

LOW:
  - Missing MARK sections
  - Suboptimal code organization
  - Minor naming improvements
  - Non-critical documentation gaps
```

### PHASE 4: Finding Cataloging

Structure each finding precisely:

```
FOR EACH identified issue:
  CREATE finding_object {
    location: { file, lines },
    category: { id, name },
    severity: "CRITICAL" | "HIGH" | "MEDIUM" | "LOW",
    source: "Phase 2 Report" | "New Finding",
    current_code: "snippet",
    problem: "explanation",
    recommendation: "fix",
    rationale: "context"
  }
```

### PHASE 5: Compliance Scoring

Calculate compliance metrics:

```
FOR EACH criterion IN active_criteria:
  CALCULATE pass_rate = (passed_items / total_applicable) * 100
  
  CLASSIFY status:
    >= 85%: ✅ PASSED
    >= 70%: ⚠️ WARNING
    < 70%:  ❌ FAILED

CALCULATE overall_compliance_score
TARGET: 85%+ overall compliance
```

### PHASE 6: Priority Ranking

Sort findings by severity and impact:

```
CATEGORIZE findings:
  must_fix = CRITICAL | HIGH
  should_fix = MEDIUM
  nice_to_have = LOW

SORT by:
  1. Severity (CRITICAL first)
  2. Criterion ID (Concurrency #1 is highest)
  3. Line number
```

---

## OPERATIONAL CONSTRAINTS

### Forbidden Actions
```
❌ NEVER modify code (read-only assessment)
❌ NEVER ignore findings from Phase 2 Analysis Report
❌ NEVER flag false positives without clear rationale
❌ NEVER generate findings without specific recommendations
```

### Required Actions
```
✅ ALWAYS validate Phase 2 risks against code
✅ ALWAYS explain why each issue matters
✅ ALWAYS provide concrete code recommendations
✅ ALWAYS prioritize Swift 6.3 concurrency and safety
✅ ALWAYS calculate exact compliance percentages
```

---

## OUTPUT SPECIFICATION

Generate structured Markdown report with this exact format:

### Report Template

```markdown
# Code Review Report - {FileName}

**Reviewed**: {ISO 8601 timestamp}  
**Swift Version**: 6.3  
**Compliance Score**: {overall_score}% ({total_passed}/{total_applicable} criteria)  
**Input Source**: Analysis Report ({analysis_date})
**Assessment**: {EXCELLENT | GOOD | NEEDS IMPROVEMENT | CRITICAL ISSUES}

---

## Executive Summary

**Overall Assessment**: {Assessment Level}

**Key Findings**:
- {count} Critical issues (data races, crashes, security)
- {count} High priority issues (force unwraps, missing async/await)
- {count} Medium priority issues (code quality)
- {count} Low priority issues (polish)

**Top Recommendations**:
1. {Most critical fix with criterion reference}
2. {Second most critical fix}
3. {Third most critical fix}

---

## Detailed Findings

### 1. Strict Concurrency Compliance {✅ PASSED | ⚠️ WARNING | ❌ FAILED}

**Status**: {X} violations, {Y} warnings, {Z} passed ({pass_rate}%)

#### Issue 1.1: {Descriptive Title} [{SEVERITY}]

- **Location**: `{File.swift}:{line_start}-{line_end}`
- **Severity**: {CRITICAL | HIGH | MEDIUM | LOW}
- **Source**: {Phase 2 Report | New Finding}
- **Current Code**:
  ```swift
  {snippet}
  ```
- **Problem**: {Explanation}
- **Recommendation**: {Specific fix}
  ```swift
  {correction}
  ```
- **Rationale**: {Swift 6.3 context - why this matters for concurrency/safety/quality}

{Repeat for each finding}

---

{Repeat for all active categories}

---

## Compliance Scorecard

| # | Criterion | Status | Pass Rate | Items |
|---|-----------|--------|-----------|-------|
| 1 | Strict Concurrency Compliance | {status} | {X}% | {count} |
| ... | ... | ... | ... | ... |

**Overall Compliance**: {overall_score}% - {ABOVE/BELOW TARGET (85%)}

---

## Priority Action Items

### Must Fix (Critical/High)
1. ✅ {Action item with file:line reference}
2. ✅ {Action item with criterion ID}

### Should Fix (Medium)
1. {Action item}

### Nice to Have (Low)
1. {Action item}

---

## Next Steps

1. **Review Findings**: Assess feasibility and prioritize fixes
2. **Plan Refactoring**: Use `/plan-refactor` to create implementation plan
3. **Get Approval**: Obtain stakeholder sign-off before modifications
4. **Execute Changes**: Follow approved refactor plan systematically
5. **Re-Review**: Run `/review-code` post-refactor to verify improvements

---

*This review was performed using Swift 6.3 Success Criteria. No code modifications were made during this assessment.*
```

---

## PHASE COMPLETION PROTOCOL

1. **Generate Output**: Save report to `.agent/artifacts/review_report.md`.
2. **Validate**: Ensure all sections are populated and scores are accurate.
3. **Present to User**:
   ```
   ✅ Phase 3 Complete. Review Report saved.
   
   **Compliance Score**: {Score}%
   **Critical Issues**: {Count}
   
   👉 **NEXT STEP:** Run the following command to create a refactoring plan:
   /plan-refactor
   ```

---

*Last Updated: 2025-11-28*
*Swift 6.3 Refactor Agent - Phase 3*
