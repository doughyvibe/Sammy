# Swift 6.3 Refactoring Agent: User Guide

Welcome to the **Sammy Swift Refactoring Agent**. This workflow is designed to assist developers in modernizing Swift codebases to **Swift 6.3** standards, with a primary focus on **Strict Concurrency**, safety, and governance compliance.

---

## 🚀 Getting Started

1. **Clone the repository**
   ```bash
   git clone https://github.com/your-org/sammy-refactor-agent.git
   cd sammy-refactor-agent
   ```
2. **Open the project** in Xcode (or SwiftPM) and ensure the `.agent` directory is visible.
3. **Configure your IDE for slash commands** (optional but recommended):
   - **Xcode**: Preferences → Behaviors → Add a custom script to run terminal commands.
   - **VS Code**: Install a ChatGPT/AI extension and map `/master-refactor`, `/context-intake`, etc., to the integrated terminal.
4. **Run the master command** to start the workflow:
   ```markdown
   ### **`/master-refactor`**
   ```

To begin a refactoring session, open the Swift file you wish to modify and run the master command:

### **`/master-refactor`**

This activates **The Architect**, your AI guide who will orchestrate the entire process.

---

## 🔄 The Workflow

```text
+---------------------------+
|  START: /master-refactor  |
+---------------------------+
             |
             v
+-----------------------------+       +-----------------------------+
|  PHASE 1: /context-intake   | ----> |  PHASE 2: /analyze-context  |
|  In: Project Files          |       |  In: Context Summary        |
|  Act: Gather Context        |       |  Act: Deep Analysis         |
|  Out: Context Summary       |       |  Out: Risk Report           |
+-----------------------------+       +-----------------------------+
                                                   |
                                                   v
+-----------------------------+       +-----------------------------+
|  PHASE 4: /plan-refactor    | <---- |  PHASE 3: /review-code      |
|  In: Issues List            |       |  In: Risk Report            |
|  Act: Create Plan           |       |  Act: Identify Issues       |
|  Out: refactor_plan.md      |       |  Out: Issues List           |
+-----------------------------+       +-----------------------------+
             |
             v
      [ USER APPROVAL ]
             |
             v
+-----------------------------+       +-----------------------------+
|  PHASE 5: /apply-refactor   | ----> |  PHASE 6: /validate-changes |
|  In: Approved Plan          |       |  In: Execution Log          |
|  Act: Apply Edits           |       |  Act: Verify Quality        |
|  Out: execution_log.md      |       |  Out: validation_report.md  |
+-----------------------------+       +-----------------------------+
                                                   |
                                           [ PASS / FAIL ]
                                                   |
                                        (Pass)     v     (Fail)
+-----------------------------+       +-----------------------------+
|  PHASE 7: /explain-changes  |       |      STOP & ROLLBACK      |
|  In: Validation Report      |       |  Review Validation Report   |
|  Act: Document Changes      |       |  for Fixes or Revert        |
|  Out: refactor_explanation  |       +-----------------------------+
+-----------------------------+
```

The process follows a strict 7-phase lifecycle to ensure safety and correctness. Do not skip steps.

### **Phase 1: Context Intake**
*   **Command**: `/context-intake @{TargetFile}`
*   **Goal**: The agent scans your project structure, governance documents (`ENGINEERING_STANDARDS.md`), and dependencies.
*   **Output**: A baseline context summary saved to `.agent/artifacts/context_summary.md`.

### **Phase 2: Deep Analysis**
*   **Command**: `/analyze-context @.agent/artifacts/context_summary.md`
*   **Goal**: Identifies concurrency risks, dependency chains, and architectural patterns in the target file.
*   **Output**: A logic and dependency report saved to `.agent/artifacts/analysis_report.md`.

### **Phase 3: Code Review**
*   **Command**: `/review-code @.agent/artifacts/analysis_report.md`
*   **Goal**: Detects specific issues, technical debt, and Swift 6.3 violations (e.g., non-Sendable types).
*   **Output**: A list of issues and improvement suggestions saved to `.agent/artifacts/review_report.md`.

### **Phase 4: Strategic Planning**
*   **Command**: `/plan-refactor @.agent/artifacts/review_report.md`
*   **Goal**: Generates a step-by-step implementation plan. **You must approve this plan** before code is touched.
*   **Output**: `.agent/artifacts/refactor_plan.md`

### **Phase 5: Execution**
*   **Command**: `/apply-refactor @.agent/artifacts/refactor_plan.md`
*   **Goal**: Applies the approved changes incrementally. It performs atomic edits and verifies syntax.
*   **Output**: Source code updates and `.agent/artifacts/refactor_execution_log.md`.

### **Phase 6: Validation**
*   **Command**: `/validate-changes @.agent/artifacts/refactor_execution_log.md`
*   **Goal**: Runs strict quality gates: build checks, concurrency validation, tests, and compliance verification.
*   **Output**: `.agent/artifacts/validation_report.md`

### **Phase 7: Explanation**
*   **Command**: `/explain-changes @.agent/artifacts/validation_report.md`
*   **Goal**: Generates final documentation explaining *what* changed and *why*, tailored to your audience.
*   **Output**: `.agent/artifacts/refactor_explanation.md`

---

## 🛡️ Core Principles

This agent operates under strict **Scope Constraints** to protect your codebase:

1.  **Safety First**: We never refactor blindly. Analysis and Planning always precede Execution.
2.  **Swift 6.3 Strict**: Zero tolerance for data races. All concurrency changes are validated against strict compiler flags.
3.  **Behavior Preservation**: The goal is refactoring, not rewriting. Business logic is preserved; structure is improved.
4.  **Human in the Loop**: You approve the plan. You review the validation. You are in control.

---

## 📂 Key Artifacts

The agent maintains its state in the `.agent/artifacts/` directory:

| File | Purpose |
|------|---------|
| `refactor_plan.md` | The approved roadmap for changes. |
| `refactor_execution_log.md` | A history of every edit applied. |
| `validation_report.md` | Pass/Fail results for build, tests, and compliance. |
| `refactor_explanation.md` | Final human-readable documentation. |

---

## 💡 Pro Tips

*   **Start Small**: Refactor one file or a small group of coupled files at a time.
*   **Trust the Validation**: If `/validate-changes` reports a failure, **do not proceed**. Roll back or fix the issue.
*   **Read the Plan**: The `/plan-refactor` step is your best chance to catch architectural misunderstandings before code is written.

---

*Ready to modernize? Open a file and type `/master-refactor`.*
