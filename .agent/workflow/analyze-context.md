# analyze-context

**Workflow Phase**: Phase 2 (Code Understanding & Analysis)  
**Target**: Swift 6.3 Refactoring Projects
**Input**: `.agent/artifacts/context_summary.md` (from Phase 1)

---

## AGENT DIRECTIVE

Execute deep static analysis on the target code identified in Phase 1. You are "The Architect" analyzing the code for Swift 6.3 compliance, logic flow, and technical debt.

**Core Responsibilities:**
1. **Ingest** the Context Summary from Phase 1.
2. **Analyze** logic, data flow, and concurrency safety (Swift 6.3).
3. **Catalog** technical debt and modernization opportunities.
4. **Produce** a detailed `Logic & Dependency Report`.

**Critical Constraint**: ❌ ZERO code modifications permitted during this phase.

---

## INPUT PARAMETERS

### Required Inputs
```
1. Context Summary Artifact:
   Path: .agent/artifacts/context_summary.md
   Content: Target files, dependencies, risk assessment, project foundation.

2. Source Code:
   Access: Read files listed in context_summary.md > target_files
```

---

## EXECUTION PROTOCOL

### STEP 1: Ingest Context Summary
```
ACTION: READ .agent/artifacts/context_summary.md
PARSE:
  - Target Files List
  - Swift Version & Concurrency Mode
  - Identified Dependencies
  - Risk Assessment
  - Architecture Pattern

VERIFY:
  - Are all target files accessible?
  - Is the context summary complete (no missing sections)?
  
IF critical gaps found in summary:
  STOP and ask user to re-run Phase 1 (/context-intake).
```

### STEP 2: Swift 6.3 Concurrency Audit
```
FOCUS: Strict Concurrency Checking

FOR EACH target file:
  1. SENDABLE ANALYSIS:
     - Identify types crossing actor boundaries.
     - Check for `Sendable` conformance.
     - Flag missing conformances or `@unchecked` usage without justification.
  
  2. ACTOR ISOLATION:
     - Check mutable state (vars).
     - Verify protection by `actor` or `@MainActor`.
     - Flag global/static mutable state (High Risk).
  
  3. ASYNC/AWAIT PATTERNS:
     - Identify legacy completion handlers.
     - Identify `DispatchQueue` usage.
     - Identify `NSLock` / `semaphore` usage.
     - Flag blocking code on main thread.

OUTPUT: Populate `concurrency_analysis` section.
```

### STEP 3: Logic & Data Flow Analysis
```
FOCUS: Code Correctness and Complexity

FOR EACH target file:
  1. CONTROL FLOW:
     - Analyze cyclomatic complexity (nested ifs, loops).
     - Flag functions > 50 lines or complexity > 10.
     - Identify force unwraps (`!`) and `try!`.
  
  2. DATA FLOW:
     - Trace data inputs to outputs.
     - Identify state mutations.
     - Check for potential retain cycles (capture lists in closures).
  
  3. ERROR HANDLING:
     - Analyze error propagation.
     - Flag swallowed errors (`try?`, empty catch).
     - Verify error types are specific (not just `Error`).

OUTPUT: Populate `logic_analysis` section.
```

### STEP 4: Dependency & Impact Verification
```
FOCUS: Blast Radius Confirmation

REFER to Dependency Map from Phase 1.
VERIFY:
  - Are all incoming calls identified?
  - Are public API changes required?
  - Will changes break external consumers?

AUGMENT:
  - Add specific line numbers for call sites if missing.
  - Classify impact of potential changes (Breaking/Non-Breaking).

OUTPUT: Populate `impact_analysis` section.
```

---

## OUTPUT SPECIFICATION

Generate a **Logic & Dependency Report** in Markdown format.

**Filename**: `.agent/artifacts/analysis_report.md`

### Template

```markdown
---
phase: 2
phase_name: Code Understanding
analysis_date: {ISO 8601}
source_context: .agent/artifacts/context_summary.md
---

# Logic & Dependency Report

## 1. Swift 6.3 Concurrency Analysis
### 🔴 High Risk (Must Fix)
- **Global Mutable State**: `{File}:{Line}` - {Description}
- **Missing Sendable**: `{Type}` - Crosses boundary without conformance
- **Data Races**: `{Location}` - Potential race condition

### 🟡 Medium Risk (Should Fix)
- **@unchecked Sendable**: `{Type}` - Missing justification
- **MainActor Missing**: `{UI Component}` - Not isolated

### 🟢 Modernization Opportunities
- **Completion Handlers**: `{Method}` -> Convert to async/await
- **DispatchQueue**: `{Location}` -> Convert to Task

## 2. Logic & Code Quality
### Complexity Hotspots
- `{Function}`: Complexity {Score}, Lines {Count}

### Safety Violations
- **Force Unwraps**: {Count} instances (Lines: {List})
- **Force Try**: {Count} instances
- **Retain Cycles**: {Count} potential cycles

## 3. Impact Analysis
### Public API Surface
- **Methods**: {Count}
- **Properties**: {Count}
- **Breaking Change Risk**: {High/Medium/Low}

### Dependency Graph
- **Incoming**: {Count} consumers
- **Outgoing**: {Count} dependencies

## 4. Technical Debt Catalog
- {Item 1}
- {Item 2}

## 5. Phase 3 Readiness
- [ ] Concurrency risks mapped
- [ ] Logic issues identified
- [ ] Impact radius confirmed

**Recommendation**: {Proceed to Planning | Request More Context}
```

---

## PHASE COMPLETION PROTOCOL

1. **Generate Output**: Save report to `.agent/artifacts/analysis_report.md`.
2. **Validate**: Ensure all sections are populated and reference Phase 1 data.
3. **Present to User**:
   ```
   ✅ Phase 2 Complete. Analysis Report saved.
   
   **Key Findings:**
   - 🔴 Critical Concurrency Issues: {Count}
   - 🟡 Modernization Opportunities: {Count}
   - 📉 Technical Debt Items: {Count}
   
   👉 **NEXT STEP:** Run the following command to review findings:
   /review-code
   ```

---

## OPERATIONAL CONSTRAINTS

- **Read-Only**: Do not modify any code.
- **Strict Concurrency**: Assume Swift 6.3 strict mode is enabled.
- **Evidence-Based**: Only report issues found in the code.
- **Dependency**: Must rely on `context_summary.md` for scope.

*Last Updated: 2025-11-28*
*Swift 6.3 Refactor Agent - Phase 2*
