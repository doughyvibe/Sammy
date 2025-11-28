# apply-refactor

**Workflow Phase**: Phase 5 (Refactor Execution)  
**Target**: Swift 6.3 Code Refactor Executor (Incremental Diff-Based Edits)
**Input**: `.agent/artifacts/refactor_plan.md` (from Phase 4)

---

## AGENT DIRECTIVE

Execute approved refactor plans by producing precise code edits as unified diffs in small, reviewable chunks. This tool implements changes incrementally while strictly preserving existing behavior and adhering to Swift 6.3 best practices.

**CRITICAL**: You must ingest the **Refactor Plan** and execute changes in the defined sequence.

---

## INPUT PARAMETERS

### Required Inputs
```
1. Refactor Plan Artifact:
   Path: .agent/artifacts/refactor_plan.md
   Content: Sequenced changes, testing strategy, rollback procedures.

2. Target Files:
   Access: Read files identified in the current change specification.

3. Scope Constraints:
   Path: .agent/core/scope-constraints.md
   Content: Forbidden actions and operational boundaries.
```

### Optional Inputs
```
{change_id}         : Specific change ID to execute (e.g., "CHANGE-001")
                      Default: "next" (executes the next pending change in sequence)
{approval_status}   : User approval confirmation ("approved" required to proceed)
```

---

## EXECUTION PROTOCOL

### PHASE 1: Ingest Refactor Plan

```
ACTION: READ .agent/artifacts/refactor_plan.md

EXTRACT:
  - Sequenced Changes (List of CHANGE-XXX)
  - Change Specifications (Files, Type, Complexity, Risk)
  - Testing Strategy (Per-change validation steps)
  - Rollback Procedures (For high-risk changes)

VERIFY:
  - Plan contains "I approve this refactor plan" checked by user?
  - If NOT approved: STOP and request user approval.
```

### PHASE 2: Select Change to Execute

```
IF {change_id} specified:
  SELECT change matching {change_id}
ELSE:
  FIND first change where status != "COMPLETED"
  SELECT that change

VALIDATE DEPENDENCIES:
  - Check if all dependencies for selected change are COMPLETED.
  - If dependencies missing: STOP and report blocking dependency.
```

### PHASE 3: Analyze Current Code State

For each file in the selected change:

```
1. READ current file contents completely.
2. LOCATE the code block identified in `Current Code` section of the plan.
3. VERIFY that the code matches the plan's expectation.
   - IF MISMATCH: STOP. Report "Drift Detected". Request manual intervention.
4. CHECK for conflicts with recent edits.
```

### PHASE 4: Generate Minimal Diff

Create a unified diff that implements the `Proposed Change` from the plan.

**Constraints**:
- ❌ **NO full file rewrites** - Only modify necessary lines
- ❌ **NO unrelated changes** - Each diff addresses exactly one change from plan
- ❌ **NO style-only changes** - Unless explicitly in the plan
- ✅ **MUST preserve** all behavior not explicitly targeted
- ✅ **MUST maintain** existing code formatting and style

**Diff Format**: Unified diff with 3 lines of context

```diff
--- a/path/to/File.swift
+++ b/path/to/File.swift
@@ -67,10 +67,8 @@
      }
      
-    func fetchUser(completion: @escaping (User?) -> Void) {
-        DispatchQueue.global().async {
-            let user = self.networkAPI.getUser()
-            DispatchQueue.main.async {
-                completion(user)
-            }
+    func fetchUser() async throws -> User {
+        return try await networkAPI.getUser()
+    }
+    
+    @available(*, deprecated, message: "Use fetchUser() async instead")
+    func fetchUser(completion: @escaping (User?) -> Void) {
+        Task {
+            let user = try? await fetchUser()
+            completion(user)
         }
     }
```

### PHASE 5: Apply Swift 6.3 Modernization Patterns

Apply specific patterns based on the Change Type:

**Concurrency Safety** (Priority 1):
- Add `Sendable` conformance to types crossing concurrency boundaries
  ```swift
  // Flag non-Sendable types in Task, actor methods, async calls
  struct User: Codable, Sendable { // ✅ Add Sendable
      let id: String
      let name: String
  }
  ```
- Add `@MainActor` to UI-touching code
  ```swift
  @MainActor
  class UserViewModel: ObservableObject {
      @Published var user: User?
  }
  ```
- Wrap shared mutable state in `actor`
  ```swift
  actor UserCache {
      private var cache: [String: User] = [:]
      func get(_ id: String) -> User? { cache[id] }
      func set(_ id: String, user: User) { cache[id] = user }
  }
  ```
- Add inline comment for `@unchecked Sendable` (ONLY if absolutely necessary)
  ```swift
  @unchecked Sendable // Safe: Immutable after init, all properties are Sendable
  class Configuration {
      let apiKey: String
      init(apiKey: String) { self.apiKey = apiKey }
  }
  ```

**Async/Await Conversion** (Priority 2):
- Replace completion handlers with `async throws`
- Use `Task` for bridging sync to async
- Add `Task.checkCancellation()` in loops or long operations
- Use `async let` for parallel independent tasks
- Use `withTaskGroup` for dynamic parallel tasks

**Optional Safety** (Priority 3):
- Replace force unwraps (`!`) with safe unwrapping
  ```diff
  - let user = getUser()!
  + guard let user = getUser() else {
  +     throw UserError.notFound
  + }
  ```
- Document remaining force unwraps with inline comment
  ```swift
  @IBOutlet var label: UILabel! // Safe: Always set by storyboard
  ```

**Error Handling** (Priority 4):
- Replace `try?` and `try!` with proper `do-catch` or `guard`
- Create custom error enums
  ```swift
  enum NetworkError: Error {
      case invalidURL
      case decodingFailed(underlying: Error)
      case serverError(statusCode: Int)
  }
  ```

### PHASE 6: Validate Edit Safety

**Before applying diff, verify**:
- ⚠️ **NO changes to public API signatures** (unless explicitly approved and deprecated)
- ⚠️ **NO changes to business logic** (only structural refactoring)
- ⚠️ **NO changes to UI/UX behavior** (colors, layouts, animations)
- ⚠️ **NO performance degradation** (no O(1) → O(n) changes)
- ⚠️ **NO removal of code** without understanding its purpose

**Scan for Red Flags**:
- Removing protocol methods (may break conformance)
- Changing `CodingKeys` (may break serialization)
- Modifying database model properties (requires migration)
- Altering localization string keys
- Touching security/compliance code

**IF RED FLAG DETECTED**:
- CHECK against .agent/core/scope-constraints.md for specific violation details
- STOP and report to user
- Explain the risk
- Request explicit confirmation or plan adjustment

### PHASE 7: Apply Diff and Verify Build

1. **Apply the diff** to target file(s)
2. **Immediate build check**: Compile the project
   - **IF BUILD FAILS**: 
     - Rollback changes immediately
     - Capture compiler errors
     - Report to user with error details
     - Mark change as "failed"
3. **Quick syntax check**: Ensure valid Swift syntax
4. **Concurrency check**: Run Swift 6.3 strict concurrency validation

### PHASE 8: Run Incremental Tests

**Test Strategy**:
- Run tests related to modified files (fast feedback)
- Do NOT run full test suite yet (leave for validate-changes.md)

**IF TESTS FAIL**:
- Rollback changes
- Capture test failure output
- Report to user
- Mark change as "failed"

---

## OUTPUT ARTIFACTS

### 1. Source Code (Primary)
- **Location**: In-place modification of target Swift files.
- **Action**: Apply unified diffs directly to the source code.

### 2. Execution Log (Secondary)
- **File**: `.agent/artifacts/refactor_execution_log.md`
- **Action**: APPEND the **Refactor Execution Report** to this file.


---

## OUTPUT FORMAT

Return a **refactor execution report** in Markdown with embedded JSON:

```markdown
# Refactor Execution Report

**Change ID**: {CHANGE-XXX}
**Timestamp**: {ISO 8601}
**Status**: ✅ COMPLETED | ❌ FAILED | ⚠️ ROLLED BACK

---

## Change Summary

**Type**: {Change Type}
**Files Modified**: `{File.swift}`
**Success Criteria**: #{ID} ({Name})
**Risk Level**: {Level}

**Rationale**: {From Plan}

---

## Code Changes (Unified Diff)

```diff
{Unified Diff Content}
```

**Lines Changed**:
- Added: {N}
- Removed: {M}
- Modified: {K}

---

## Verification Results

✅ **Build**: SUCCESS
✅ **Tests**: PASSED ({N}/{N} tests)
✅ **Concurrency**: PASSED (Strict Mode)

---

## Next Action

**Options**:
1. ✅ Proceed to next change: `{CHANGE-YYY}`
2. 🔍 Validate all changes so far with `/validate-changes`
3. 📋 Review execution log at .agent/artifacts/refactor_execution_log.md

**Recommendation**: Proceed to {CHANGE-YYY}
```

---

## ROLLBACK PROCEDURE

If a change fails or user requests rollback:

1. **Revert file(s)** to pre-change state (Git undo or file backup)
2. **Mark change as "rolled back"** in execution log
3. **Report rollback reason** to user
4. **Update plan status**: Mark change as "failed" or "skipped"
5. **Recommend action**: Fix issue and retry, or skip and proceed

---

## OPERATIONAL CONSTRAINTS

### Forbidden Actions
```
❌ NEVER add new features
❌ NEVER break public APIs without approval
❌ NEVER modify business logic
❌ NEVER refactor more than 3 files at once
❌ NEVER skip build verification
```

### Required Actions
```
✅ ALWAYS preserve behavior
✅ ALWAYS run tests before and after
✅ ALWAYS produce diffs (no full rewrites)
✅ ALWAYS check Sendable conformance
✅ ALWAYS maintain compatibility
```

---

*This tool executes approved refactor plans incrementally. Always verify prerequisites and run tests. Preserve behavior above all else.*

*Last Updated: 2025-11-28*
*Swift 6.3 Refactor Agent - Phase 5*
