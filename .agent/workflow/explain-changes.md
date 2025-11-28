# explain-changes

**Workflow Phase**: Phase 7 (Change Documentation)  
**Target**: Swift 6.3 Refactoring Explanation Generation

---

## AGENT DIRECTIVE

Generate comprehensive, evidence-based explanations for completed refactoring changes. Transform technical modifications into clear narratives with Success Criteria mapping, metrics, rationale, and stakeholder-appropriate insights. **CRITICAL**: All claims must be backed by concrete evidence from execution logs and validation reports—no exaggeration or speculation.

**Output**: Structured explanation document tailored to target audience

---

## OUTPUT ARTIFACTS

### 1. Explanation Document
- **File**: `.agent/artifacts/refactor_explanation.md`
- **Action**: OVERWRITE with the full **Refactoring Explanation**.
- **Purpose**: Final documentation for the user and team.

---

## INPUT PARAMETERS

### Required Inputs
```
{execution_log}      : .agent/artifacts/refactor_execution_log.md (from Phase 5)
{validation_report}  : .agent/artifacts/validation_report.md (from Phase 6)
```

### Optional Inputs
```
{original_context}   : Phase 1 context summary (for baseline comparison)
{target_audience}    : Explanation audience
                       - "technical"     : Developers (code examples, metrics) DEFAULT
                       - "management"    : Non-technical stakeholders (business focus)
                       - "documentation" : Knowledge base (balanced depth)

{detail_level}       : Content depth
                       - "summary"       : High-level overview only
                       - "detailed"      : Change-by-change analysis DEFAULT
                       - "comprehensive" : Includes code examples and patterns

{format}             : Output format
                       - "markdown"      : Human-readable document DEFAULT
                       - "json"          : Structured data
                       - "html"          : Web-ready format
```

---

## EXECUTION PROTOCOL

### PHASE 1: Data Collection & Parsing

Load and validate all input sources:

```
STEP 1.1: Load Execution Log
  
  PARSE execution_log as JSON
  
  EXTRACT:
    changes_completed = log.changes[]
    files_modified = log.files_modified[]
    timestamp = log.completed_at
    project_name = log.project_name
  
  FOR EACH change IN changes_completed:
    EXTRACT change metadata:
      - change_id
      - change_type
      - files_affected
      - lines_added
      - lines_removed
      - risk_level
      - success_criteria_refs[]
      - before_code
      - after_code
  
  GROUP changes by category:
    concurrency_changes = changes WHERE type IN ["sendable", "mainactor", "async"]
    safety_changes = changes WHERE type IN ["unwrap", "error_handling"]
    quality_changes = changes WHERE type IN ["refactor", "naming", "docs"]
    testing_changes = changes WHERE type IN ["test_addition", "dependency_injection"]

STEP 1.2: Load Validation Report
  
  PARSE validation_report
  
  EXTRACT metrics:
    - overall_validation_status
    - gate1_results (build, tests, concurrency)
    - gate2_results (unwraps, docs, compliance, complexity, coverage)
    - compliance_score (overall percentage)
    - test_results (passed, failed, added)
    - build_results (errors, warnings)
    - quality_metrics (before/after comparisons)
  
  EXTRACT Swift 6.3 validations:
    - sendable_compliance
    - mainactor_coverage
    - actor_isolation_status
    - global_state_safety

STEP 1.3: Load Original Context (If Available)
  
  IF original_context exists:
    EXTRACT:
      - user_objectives[]
      - baseline_metrics
      - initial_risk_assessment
      - project_characteristics
  
  CREATE baseline comparison capability

STEP 1.4: Calculate Aggregate Statistics
  
  COMPUTE totals:
    total_files_modified = COUNT(UNIQUE(files_modified))
    total_lines_added = SUM(changes.lines_added)
    total_lines_removed = SUM(changes.lines_removed)
    net_lines_changed = total_lines_added - total_lines_removed
    total_changes = COUNT(changes_completed)
  
  COMPUTE improvement metrics:
    force_unwrap_reduction = (baseline - current) / baseline × 100
    warning_reduction = baseline_warnings - current_warnings
    complexity_improvement = baseline_complexity - current_complexity
    coverage_improvement = current_coverage - baseline_coverage
    compliance_improvement = current_compliance - baseline_compliance
```

### PHASE 2: Success Criteria Mapping

Map each change to Success Criteria categories:

```
INITIALIZE criteria_map = {} // criterion_id -> [changes]

FOR EACH change IN changes_completed:
  EXTRACT change.success_criteria_refs[]
  
  FOR EACH criterion_id IN success_criteria_refs:
    ADD change to criteria_map[criterion_id]
    
    DETERMINE criterion contribution:
      - How many checklist items this change addresses
      - Percentage contribution to criterion compliance
      - Before/after status for this criterion

CREATE success criteria summary:
  FOR criterion_id IN [1..16]:
    IF criterion_id IN criteria_map:
      STATUS = ADDRESSED
      changes_count = COUNT(criteria_map[criterion_id])
      compliance_status = validation_report.criteria[criterion_id].status
    ELSE:
      STATUS = NOT_APPLICABLE
      changes_count = 0
```

### PHASE 3: Change Narrative Generation

Create structured explanation for each change:

```
FOR EACH change IN changes_completed:
  
  CREATE change_narrative {
    
    change_id: change.id,
    title: "{descriptive title based on type}",
    
    what_changed: "{one-sentence summary}",
      RULES:
        - Use active voice
        - Be specific (file, method, pattern)
        - Avoid jargon unless technical audience
    
    why_it_matters: "{business/technical rationale}",
      RULES:
        - Connect to user impact or code quality
        - Explain problem being solved
        - Reference Swift 6.3 context if applicable
    
    success_criteria: [
      FOR EACH criterion_id IN change.success_criteria_refs:
        "✅ #{criterion_id} - {criterion_name}"
    ],
    
    impact: {
      files: change.files_affected,
      lines_added: change.lines_added,
      lines_removed: change.lines_removed,
      risk_level: change.risk_level,
      breaking_change: change.is_breaking
    },
    
    benefit: [
      LIST concrete improvements:
        - Safety benefit (if applicable)
        - Readability benefit (if applicable)
        - Performance benefit (if applicable)
        - Maintainability benefit (if applicable)
    ],
    
    code_comparison: {
      IF detail_level IN ["detailed", "comprehensive"]:
        before: change.before_code,
        after: change.after_code,
        improvements: ["{specific improvement points}"]
    }
  }
  
  ADD change_narrative to narratives_collection
```

### PHASE 4: Thematic Grouping & Organization

Organize changes into logical categories:

```
GROUP narratives INTO themes:

THEME 1: CONCURRENCY & THREAD SAFETY
  INCLUDE:
    - Sendable conformance additions
    - @MainActor annotations
    - Actor introductions
    - Async/await conversions
  
  INTRO TEXT (if any changes):
    "Swift 6.3 introduces strict concurrency checking to eliminate 
     data races at compile-time. We made the following improvements:"

THEME 2: SAFETY IMPROVEMENTS
  INCLUDE:
    - Force unwrap eliminations
    - Optional handling improvements
    - Error handling enhancements
    - Type safety improvements
  
  INTRO TEXT (if any changes):
    "Improved code safety to prevent crashes and handle errors gracefully:"

THEME 3: CODE QUALITY & MAINTAINABILITY
  INCLUDE:
    - Function extraction/refactoring
    - Naming improvements
    - Complexity reduction
    - Code organization
  
  INTRO TEXT (if any changes):
    "Enhanced code quality for better readability and maintainability:"

THEME 4: DOCUMENTATION & TESTING
  INCLUDE:
    - Documentation additions
    - Test additions
    - Dependency injection improvements
  
  INTRO TEXT (if any changes):
    "Improved documentation and test coverage:"

SORT themes BY:
  1. Number of changes (most changes first)
  2. Impact level (highest impact first)
```

### PHASE 5: Key Achievement Identification

Select top achievements for executive summary:

```
RANK all changes BY:
  score = (
    (impact_level × 3) +         // High=3, Medium=2, Low=1
    (safety_critical × 2) +       // Boolean × 2
    (compliance_boost × 2) +      // Boolean × 2
    (complexity_reduction × 1)    // Boolean × 1
  )

SELECT top 3-5 changes WITH highest scores

FOR EACH top achievement:
  CREATE achievement {
    emoji: "{relevant emoji: 🎯🔒🚀📚✅}",
    title: "{impactful one-liner}",
    details: [
      "{quantified improvement}",
      "{criterion compliance status}",
      "{additional context}"
    ],
    metric: "{before → after with percentage}"
  }

EXAMPLE:
  {
    emoji: "🎯",
    title: "Eliminated 92% of Force Unwraps",
    details: [
      "Prevents potential crashes from network/user data",
      "Criterion #3: Safe Optional Handling now 92% compliant",
      "All remaining unwraps justified (IBOutlets only)"
    ],
    metric: "12 → 1 (-92%)"
  }
```

### PHASE 6: Rationale Summary Generation

For each addressed Success Criterion, explain WHY it matters:

```
FOR EACH criterion IN addressed_criteria:
  
  CREATE rationale_summary {
    criterion_name: "{criterion name}",
    
    technical_rationale: "{technical explanation}",
      TEMPLATE:
        "Explain the technical mechanism/problem this criterion addresses.
         Reference Swift language features, compiler behavior, or architecture."
    
    business_impact: "{user-facing or business benefit}",
      TEMPLATE:
        "Translate technical benefit to business outcomes.
         Mention: crashes prevented, user experience, development speed."
    
    swift63_context: "{Swift 6.3 specific relevance}",
      TEMPLATE:
        "How this aligns with Swift 6.3 goals and modern Apple requirements.
         Mention: strict concurrency, compilation requirements, framework compatibility."
    
    longterm_value: "{maintenance and scalability}",
      TEMPLATE:
        "Future benefits for codebase health and team velocity.
         Mention: easier refactoring, onboarding, debugging."
  }

EXAMPLE for Criterion #1 (Strict Concurrency):
  technical_rationale: "Swift 6.3's strict concurrency mode catches data races 
    at compile-time instead of runtime. Without Sendable types and proper actor 
    isolation, concurrent code can corrupt memory, causing crashes or silent 
    data corruption."
  
  business_impact: "Data races cause unpredictable bugs that are hard to 
    reproduce and debug. Strict concurrency eliminates an entire class of bugs 
    before code ships, reducing production incidents and customer frustration."
  
  swift63_context: "Apple requires strict concurrency for new apps in 2025+. 
    Adopting it now prevents future migration pain and ensures compatibility 
    with modern Swift frameworks."
  
  longterm_value: "Concurrency-safe code is easier to parallelize, optimize, 
    and maintain. Future developers can confidently modify concurrent code 
    without fear of introducing race conditions."
```

### PHASE 7: Before/After Code Comparison

For high-impact changes, generate code comparisons:

```
SELECT changes FOR comparison WHERE:
  - detail_level IN ["detailed", "comprehensive"]
  - change has before/after code
  - change is high impact OR addresses critical criterion

FOR EACH selected change:
  CREATE code_comparison {
    
    title: "{change title}",
    
    before_section: {
      label: "Before (Legacy Pattern)",
      code: change.before_code,
      problems: [
        LIST specific issues with before code:
          "❌ {problem description}"
      ]
    },
    
    after_section: {
      label: "After (Swift 6.3 Pattern)",
      code: change.after_code,
      benefits: [
        LIST specific improvements:
          "✅ {benefit description}"
      ]
    },
    
    quantified_improvements: [
      IF lines reduced:
        "📉 Lines of code: {before} → {after} ({percentage}% reduction)"
      
      IF complexity reduced:
        "🔧 Cyclomatic complexity: {before} → {after}"
      
      IF specific benefits:
        "🔒 {safety improvement}"
        "📍 {maintainability improvement}"
        "⚠️ {error handling improvement}"
        "🧪 {testability improvement}"
    ]
  }
```

### PHASE 8: Governance Compliance Documentation

Document adherence to project standards:

```
VERIFY governance compliance:

IF ENGINEERING_STANDARDS.md referenced IN original_context:
  CHECK modified code against standards:
    - Naming conventions (camelCase, PascalCase)
    - Code organization (MARK sections)
    - Architecture patterns (MVVM, etc.)
    - Approved dependency patterns
  
  REPORT compliance with specific section references

IF CODE_TEMPLATES.md referenced IN original_context:
  CHECK new patterns match templates:
    - ViewModel patterns
    - Network service patterns
    - Error handling patterns
  
  REPORT compliance with template adherence

VERIFY .agent/core/scope-constraints.md compliance:
  CONFIRM forbidden actions NOT taken:
    ✅ No new features added
    ✅ No breaking API changes (or deprecated properly)
    ✅ No business logic alterations
    ✅ No file restructuring beyond scope
    ✅ Incremental changes only (file count within limits)
    ✅ All tests passing (behavior preserved)
```

### PHASE 9: Audience-Specific Tailoring

Adjust content and tone based on target_audience:

```
IF target_audience == "technical":
  INCLUDE:
    - Detailed code examples
    - Technical terminology and references
    - Swift-specific feature explanations
    - Metrics and statistics with precision
    - Links to Swift Evolution proposals (if relevant)
  
  TONE: Professional developer-to-developer
  CODE: Full before/after snippets
  DEPTH: Comprehensive technical rationale

ELSE IF target_audience == "management":
  INCLUDE:
    - Business benefit focus
    - Risk reduction emphasis
    - Quality improvement highlights
    - Minimal code examples (high-level only)
    - Plain language explanations
  
  TONE: Business-focused, outcome-oriented
  CODE: Minimal, only for context
  DEPTH: High-level with business justification
  
  TRANSFORM technical terms:
    "Sendable conformance" → "Thread-safety guarantees"
    "Force unwrap elimination" → "Crash prevention"
    "Async/await" → "Modern asynchronous patterns"
    "Cyclomatic complexity" → "Code simplicity"

ELSE IF target_audience == "documentation":
  INCLUDE:
    - Balanced technical depth
    - Reusable pattern highlights
    - Links to external resources
    - Structured for knowledge base
    - Best practices emphasis
  
  TONE: Educational and reference-oriented
  CODE: Representative examples
  DEPTH: Instructional with context
```

### PHASE 10: Output Formatting & Generation

Generate final output in requested format:

```
IF format == "markdown":
  GENERATE using markdown template (see OUTPUT SPECIFICATION)
  
  INCLUDE all sections:
    - Executive Summary
    - Changes by Category (thematic grouping)
    - Success Criteria Mapping
    - Why These Changes Matter (rationale summaries)
    - Governance Compliance Summary
    - Quality Metrics Summary (metrics table)
    - Validation Status (from validation_report)
    - Next Steps
    - Lessons Learned & Reusable Patterns
  
  APPLY audience-specific tailoring
  APPLY detail-level filtering

ELSE IF format == "json":
  GENERATE structured JSON (see JSON SPECIFICATION)
  
  STRUCTURE:
    - refactoring_summary (metadata)
    - impact (aggregate statistics)
    - changes_by_category (grouped changes)
    - success_criteria_status (compliance per criterion)
    - metrics (before/after comparisons)
    - validation_status (gate results)
  
  ENSURE all values are machine-readable

ELSE IF format == "html":
  GENERATE HTML document
  
  WRAP markdown content in HTML template
  ADD styling and navigation
  INCLUDE interactive elements (collapsible sections)
```

---

## OPERATIONAL CONSTRAINTS

### Forbidden Actions
```
❌ NEVER exaggerate benefits or overstate improvements
❌ NEVER hide issues found during validation
❌ NEVER claim new features were added (only refactoring)
❌ NEVER blame original code or developers
❌ NEVER make vague claims without metric backing
❌ NEVER skip Success Criteria references
❌ NEVER invent code examples (use actual before/after from log)
```

### Required Actions
```
✅ ALWAYS reference Success Criteria by number (#1-#16)
✅ ALWAYS provide concrete metrics (exact before/after numbers)
✅ ALWAYS explain rationale (WHY not just WHAT)
✅ ALWAYS cite compliance with governance documents
✅ ALWAYS acknowledge tradeoffs if any exist
✅ ALWAYS include validation status from report
✅ ALWAYS base explanations on evidence from logs
✅ ALWAYS tailor content to target audience
```

---

## SWIFT 6.3 EXPLANATION PATTERNS

Pre-defined narrative patterns for common change types:

### Pattern 1: Sendable Conformance Addition

```
TEMPLATE:
  What: "Added `Sendable` conformance to `{TypeName}` {struct|class|enum}"
  
  Why: "In Swift 6.3, types passed across concurrency boundaries (like into 
    `Task {}` or async methods) must be `Sendable` to prevent data races.
    {TypeName} is now marked Sendable, guaranteeing the compiler that:
    - All properties are themselves Sendable
    - The type has no mutable state shared across threads
    - It's safe to pass between actors, tasks, and threads"
  
  Criterion: "✅ #1 - Strict Concurrency Compliance"
  
  Benefit: "Eliminates compile-time warnings and prevents potential data 
    corruption from concurrent access. Enables Swift 6.3 strict mode compilation."
```

### Pattern 2: @MainActor Application

```
TEMPLATE:
  What: "Applied `@MainActor` to `{ClassName}` class"
  
  Why: "SwiftUI requires all UI updates occur on the main thread. By marking 
    {ClassName} with @MainActor, Swift 6.3 guarantees all methods and property 
    access happen on the main thread. This eliminates:
    - Manual `DispatchQueue.main.async` calls
    - Potential main-thread checker warnings
    - Risk of UI updates from background threads (causes crashes)"
  
  Criterion: "✅ #1 - Strict Concurrency Compliance"
  
  Benefit: "Compile-time safety for UI updates. No runtime crashes from 
    accessing @Published properties from wrong threads. Purple warnings 
    eliminated in Xcode."
```

### Pattern 3: Async/Await Conversion

```
TEMPLATE:
  What: "Converted `{methodName}(completion:)` to `async throws` function"
  
  Why: "Completion handlers have several issues:
    - Retain cycles from closure captures (requires `[weak self]`)
    - Easy to forget to call completion or call it twice
    - Manual thread-hopping with DispatchQueue
    - Poor error handling (optional result instead of throwing)
    
    Async/await solves these:
    - Automatic memory safety (no `[weak self]` needed)
    - Compiler ensures completion (via return value)
    - Automatic thread management via actors
    - Natural error propagation with `throws`"
  
  Criteria: 
    "✅ #2 - Modern Async/Await Patterns"
    "✅ #8 - Memory Management (no retain cycles)"
  
  Benefit: "More readable, safer, and easier to maintain. {percentage}% less code."
```

### Pattern 4: Force Unwrap Elimination

```
TEMPLATE:
  What: "Replaced `{operator}` force unwrap with proper {technique}"
    // operator: try!, !, as!
    // technique: do-catch, guard let, if let, ??
  
  Why: "Force unwrapping crashes the app if the operation fails. {Data source}
    is untrusted and can fail for many reasons:
    - {reason 1}
    - {reason 2}
    - {reason 3}
    
    Proper error handling with {technique} provides graceful failure."
  
  Criterion: "✅ #3 - Safe Optional Handling"
  
  Benefit: "No crashes from {failure scenario}. Errors are logged and 
    recoverable. Users see graceful error messages instead of app crashes."
```

---

## OUTPUT SPECIFICATION

### Markdown Report Template

```markdown
# Refactoring Changes Explanation

**Project**: {ProjectName}  
**Refactoring Completed**: {ISO 8601 timestamp}  
**Validation Status**: {✅ PASSED | ⚠️ PASSED WITH WARNINGS | ❌ FAILED}  
**Compliance Score**: {current}% (up from {baseline}%)  
**Target Audience**: {target_audience}

---

## Executive Summary

{IF target_audience == "management":
  This refactoring improved code quality, safety, and maintainability while 
  preserving all existing functionality. All changes reduce technical risk 
  and improve development velocity going forward.
ELSE:
  This refactoring improved code quality, safety, and maintainability while 
  preserving all existing functionality. All changes align with Swift 6.3 
  best practices and pass comprehensive quality gates.
}

**Key Achievements**:
{FOR EACH achievement IN top_achievements:}
- {emoji} **{title}** ({metric})
  - {detail 1}
  - {detail 2}
  - {detail 3}

**Impact**:
- Files modified: {count}
- Lines changed: +{added}, -{removed} (net {net} lines)
- Tests: All {total} passing ({new} new tests added)
- Build: Zero errors, zero warnings
- Risk: {LOW | MEDIUM | HIGH} ({explanation})

---

## Changes by Category

{FOR EACH theme IN [concurrency, safety, quality, docs]:
  IF theme has changes:
    INCLUDE theme section
}

### {emoji} {Theme Title} ({count} changes)

{theme_intro_paragraph}

#### Change {id}: {Title}

**File**: `{relative/path/to/file.swift}`  
**Lines**: {IF added > 0: "+{added}"}{IF removed > 0: ", -{removed}"}

{IF detail_level IN ["detailed", "comprehensive"]:}

**What Changed**:
{IF has code comparison:}
```swift
-{before line}
+{after line}
```
{ELSE:}
{one-sentence description}

**Why It Matters**:
{rationale paragraph}

**Success Criterion**: {FOR EACH: ✅ #{id} - {name}}

**Technical Rationale**:
{technical explanation with Swift 6.3 context}

**Benefit**:
{FOR EACH benefit:}
- {benefit description}

{IF detail_level == "comprehensive" AND has code comparison:}

---

**Before** (Legacy Pattern):
```swift
// ❌ Problems: {problem1}, {problem2}
{full before code}
```

**After** (Swift 6.3 Pattern):
```swift
// ✅ Benefits: {benefit1}, {benefit2}
{full after code}
```

**Improvements**:
- 📉 {metric improvement}
- 🔒 {safety improvement}
- 📍 {clarity improvement}
- ⚠️ {error handling improvement}
- 🧪 {testability improvement}

---

{END IF comprehensive}
{ELSE IF detail_level == "summary":}

**{change.title}**: {one-line description} (✅ #{criterion})

{END IF}
{END FOR each change in theme}
{END FOR each theme}

---

## Success Criteria Mapping

### Overall Compliance: {percentage}% ({passed}/{applicable} criteria)

{IF target_audience == "technical":}

**Criteria Fully Satisfied** (100% compliance):
{FOR EACH criterion WITH 100% compliance:}
- ✅ #{id} - {name} ({items_passed}/{items_total} items)

**Criteria Significantly Improved** (70-99% compliance):
{FOR EACH criterion WITH 70-99% compliance:}
- ⚠️ #{id} - {name}: {percentage}% ({items_passed}/{items_total})
  - {explanation of remaining items}

**Criteria Not Applicable**:
{FOR EACH N/A criterion:}
- ➖ #{id} - {name} ({reason why not applicable})

{ELSE:}

All code quality standards checked and validated. See detailed compliance 
report for technical breakdown.

{END IF}

---

## Why These Changes Matter

{FOR EACH criterion IN addressed_criteria:}

### {Criterion Name}

**{IF audience == "management": Business Impact ELSE: Technical Rationale}**:  
{audience-appropriate explanation}

{IF audience == "technical":}
**Swift 6.3 Context**: {Swift evolution context}

**Long-term Value**: {maintenance and scalability benefits}
{END IF}

{END FOR}

---

{IF audience == "technical":}

## Governance Compliance Summary

### ✅ ENGINEERING_STANDARDS.md - COMPLIANT
{FOR EACH standard section:}
- {Section}: {compliance description}

### ✅ CODE_TEMPLATES.md - COMPLIANT
{FOR EACH template:}
- {Template name}: {usage description}

### ✅ scope-constraints.md - NO VIOLATIONS
{FOR EACH constraint:}
- ✅ {constraint description}

{END IF}

---

## Quality Metrics Summary

| Metric | Before | After | Change | Target | Status |
|--------|--------|-------|--------|--------|--------|
| Force Unwraps | {before} | {after} | {percentage}% | -80% | {✅\u2705⚠❌} |
| Compiler Warnings | {before} | {after} | {change} | 0 | {status} |
| Data Race Warnings | {before} | {after} | {change} | 0 | {status} |
| Test Coverage | {before}% | {after}% | {change:+}% | No decrease | {status} |
| Avg Complexity | {before} | {after} | {change} | <10 | {status} |
| Avg Function Length | {before} | {after} | {change} lines | <50 | {status} |
| Public API Docs | {before}% | {after}% | {change:+}% | 100% | {status} |
| Checklist Compliance | {before}% | {after}% | {change:+}% | 85% | {status} |

---

## Validation Status

{✅ | ⚠️ | ❌} **{OVERALL STATUS}**

{IF technical audience:}
- {✅ | ❌} Build: {STATUS} ({errors} errors, {warnings} warnings)
- {✅ | ❌} Tests: {passed}/{total} PASSED ({percentage}%)
- {✅ | ❌} Swift 6.3 Strict Concurrency: {STATUS}
- {✅ | ❌} Compliance Score: {percentage}% (target: 85%+)

*Full validation report available in validate-changes.md output*
{ELSE:}
All automated quality checks passed. Code meets production standards.
{END IF}

---

## Next Steps

{IF validation passed:}
1. ✅ **Phase 7 Complete**: Changes documented and explained
2. 📋 **Review with team**: Share this explanation for team awareness
3. 🚀 **Merge to main**: All quality gates passed, ready for production
4. 📊 **Monitor metrics**: Track any performance or behavior changes post-deploy
5. 📖 **Update documentation**: Incorporate patterns into team knowledge base
{ELSE:}
1. ⚠️ **Address warnings**: Review validation warnings before merge
2. 📋 **Team discussion**: Discuss tradeoffs and acceptance criteria
3. 🔄 **Iterate if needed**: Optional improvements identified
{END IF}

---

{IF detail_level == "comprehensive":}

## Lessons Learned & Reusable Patterns

### Pattern 1: {Pattern Name}
{reusable steps and guidance}

### Pattern 2: {Pattern Name}
{reusable steps and guidance}

{Pattern examples from changes}

---

{END IF}

*This explanation generated from execution log and validation report. All 
changes are behavior-preserving and meet Swift 6.3 best practices.*
```

---

## JSON OUTPUT SPECIFICATION

```json
{
  "refactoring_summary": {
    "project": "string",
    "completed": "ISO 8601 timestamp",
    "validation_status": "passed" | "passed_with_warnings" | "failed",
    "compliance_score": 0.0-1.0,
    "compliance_improvement": 0.0-1.0,
    "target_audience": "technical" | "management" | "documentation"
  },
  "impact": {
    "files_modified": number,
    "lines_added": number,
    "lines_removed": number,
    "net_change": number,
    "tests_passing": number,
    "tests_added": number,
    "risk_level": "low" | "medium" | "high"
  },
  "key_achievements": [
    {
      "emoji": "string",
      "title": "string",
      "metric": "string (before → after)",
      "details": ["string"]
    }
  ],
  "changes_by_category": {
    "concurrency": [...],
    "safety": [...],
    "quality": [...],
    "documentation": [...]
  },
  "change_schema": {
    "id": "string",
    "title": "string",
    "file": "string",
    "lines_added": number,
    "lines_removed": number,
    "success_criteria": [number],
    "impact_level": "low" | "medium" | "high",
    "benefit": "string"
  },
  "success_criteria_status": [
    {
      "id": 1-16,
      "name": "string",
      "status": "passed" | "warning" | "failed" | "n/a",
      "compliance": 0.0-1.0,
      "items_passed": number,
      "items_total": number
    }
  ],
  "metrics": {
    "force_unwraps": {"before": number, "after": number, "reduction": 0.0-1.0},
    "warnings": {"before": number, "after": number, "reduction": number},
    "coverage": {"before": 0.0-1.0, "after": 0.0-1.0, "improvement": 0.0-1.0},
    "complexity": {"before": number, "after": number, "reduction": 0.0-1.0}
  },
  "validation_summary": {
    "gate1_passed": boolean,
    "gate2_passed": boolean,
    "build_status": "success" | "failed",
    "test_status": "passed" | "failed",
    "concurrency_clean": boolean
  }
}
```

---

## USAGE EXAMPLES

### Example 1: Technical Audience, Comprehensive Detail
```
User: /explain-changes target_audience="technical" detail_level="comprehensive"
Agent: [Generates detailed technical explanation with:
        - Full code before/after examples
        - Detailed Swift 6.3 rationale
        - Metrics and statistics
        - Success criteria mapping with detailed compliance
        - Governance document references
        - Reusable pattern extraction]
```

### Example 2: Management Audience, Summary Level
```
User: /explain-changes target_audience="management" detail_level="summary"
Agent: [Generates business-focused summary with:
        - High-level achievements (risk reduction, quality improvement)
        - Minimal technical jargon
        - Business impact emphasis
        - Quality metrics in plain language
        - No code examples (except high-level diagrams if helpful)]
```

### Example 3: JSON Export for Documentation System
```
User: /explain-changes format="json"
Agent: [Outputs structured JSON with:
        - Machine-readable change data
        - Categorized improvements
        - Metrics as numerical values
        - Success criteria compliance data
        - Suitable for documentation generation systems]
```

---

## CRITICAL OPERATIONAL PRINCIPLES

1. **Evidence-Based Claims**: Every statement backed by execution log or validation report
2. **Audience Awareness**: Tailor complexity and terminology to target audience
3. **Success Criteria Focus**: Always map changes to specific criteria
4. **Honest Assessment**: Acknowledge warnings and tradeoffs, don't hide issues
5. **Concrete Metrics**: Provide exact numbers, not vague improvements
6. **Rationale Over Description**: Explain WHY changes matter, not just WHAT changed
7. **Pattern Recognition**: Identify reusable patterns for team knowledge

---

*This tool generates clear, evidence-based explanations with Success Criteria mapping, metrics, and rationale. All explanations derived from execution logs and validation reports—no speculation or exaggeration.*

*Last Updated: 2025-11-26*
