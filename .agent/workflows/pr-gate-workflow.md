---
description: Standardized PR Gate Workflow between Spec Generation and Gate 2 Approval covering parallel spec execution, reviewer detection after git pull, role decision prompts, reviewer identity validation, Gate 1 & Gate 2 standardized review templates, and dashboard synchronization.
---

# PR Gate Workflow (Spec Generation → Gate 1 → Development → Gate 2)

> [!IMPORTANT]
> **Scope Restriction**
> This workflow governs exclusively the PR Gate lifecycle between **Spec Generation and Gate 2 Approval**.
> Workflows before Spec Generation (BRD, Setup, Ingestion) and after Gate 2 Approval (Release Management, CRs, Hotfixes, Deployment) remain 100% unchanged.

---

## 1. Non-Blocking Parallel Spec Execution

Multiple specs exist and progress independently. Specs in different lifecycle stages do not block one another:
- `spec-1` → Waiting for Gate 1 (`In Peer Review`)
- `spec-2` → Waiting for Gate 1 (`In Peer Review`)
- `spec-3` → Gate 1 Approved (`Approved`)
- `spec-4` → In Development (`Under Development`)
- `spec-5` → Waiting for Gate 2 (`In QA`)

---

## 2. Reviewer Detection & Trigger Mechanism

The PR Gate Workflow supports **Hybrid Triggering** for optimal user experience:

### A. Context-Aware Session Resume / Git Pull Notification
When a user pulls code or resumes an agent session:
1. The agent inspects Git user credentials: `git config user.name`, `git config user.email` (or configured User ID).
2. The agent queries `.ai-context/dashboard.html` and `.ai-context/specs/` for assigned pending reviews.
3. If pending reviews exist for this user identity, the agent displays a non-blocking notification:
   > 📌 **Pending PR Reviews**: You have **N pending reviews** assigned to you.
   > *Type **`/pr-gate-workflow`** to launch the reviewer workspace, or proceed with your command.*

### B. On-Demand Execution (`/pr-gate-workflow`)
When the user types **`/pr-gate-workflow`** (or selects a PR review prompt), the agent presents the Decision Prompt:

> **"You are assigned as a PR reviewer. What would you like to do?"**
> - **[ Option 1 — Review Pending Specs ]**
> - **[ Option 2 — Work on Approved Specs ]**

---

## 3. Option 1 — Review Pending Specs Protocol (Complete Lifecycle)

1. **Role-Based Spec Filtering & Listing**:
   The agent queries `.ai-context/specs/` and `.ai-context/dashboard.html` for specs waiting for review where current user identity matches the assigned reviewer roster:
   - **Gate 1 Approver**: Shows pending Gate 1 Spec Peer Reviews (`In Peer Review`).
   - **Gate 2 Approver**: Shows pending Gate 2 Code Reviews (`In QA`).
   - **Dual Approver (Gate 1 & Gate 2)**: Shows BOTH Gate 1 and Gate 2 pending reviewals in the listing!

   > **"Which spec would you like to review?"**

   | # | Spec ID | Spec Title | Gate Level | Assigned Role | Developer | Current Status |
   |---|---|---|---|---|---|---|
   | 1 | `auth-service` | User Authentication | **Gate 1** | Gate 1 Reviewer | Dev A | Pending Spec Review |
   | 2 | `payment-gateway` | Payment Gateway | **Gate 2** | Gate 2 Reviewer | Dev B | Pending Code Review |
   | 3 | `order-engine` | Order Processing | **Gate 1 & Gate 2** | Dual Reviewer | Dev C | Pending Spec Review |

2. **Spec-Oriented Interactive Q&A Review Process**:
   When the reviewer selects a spec:
   - **Step A — Spec Orientation Summary**: Agent displays feature Intent, Linked BRD, Acceptance Criteria, and (for Gate 2) code diff summary & test suite results.
   - **Step B — Interactive Q&A Criteria Evaluation**: Agent collects criteria scores across the 11 Gate 1 or Gate 2 metrics.
   - **Step C — Review Description**: Agent prompts for high-level findings summary.
   - **Step D — Review Comments**: Agent prompts for detailed line-item notes or requested changes.
   - **Step E — Final Decision**: Agent prompts for decision: `Approve`, `Reject`, or `Changes Requested`.

3. **Dedicated Review Record Artifact File Creation**:
   Upon review completion, automatically write a dedicated review artifact file under:
   - `.ai-context/pr_reviews/GATE1-<feature-slug>-<YYYYMMDD-HHMMSS>.md` (for Gate 1)
   - `.ai-context/pr_reviews/GATE2-<feature-slug>-<YYYYMMDD-HHMMSS>.md` (for Gate 2)

4. **Git Identity Validation**:
   Validate authenticated Git credentials match the assigned reviewer roster before committing the review outcome.

5. **5-Artifact Synchronization Protocol**:
   Synchronize review completion across **5 core repository artifacts**:
   - **1. Dedicated PR Review File**: Saved under `.ai-context/pr_reviews/`.
   - **2. Dashboard HTML (`.ai-context/dashboard.html` — SINGLE SOURCE OF TRUTH)**: Updated with complete 22-field template, criteria scores, comments, description, reviewer ID, timestamp.
   - **3. Governing Spec (`.ai-context/specs/<slug>.spec.md`)**: Updated status and `## Gate Approvals & History` log linking to review file.
   - **4. Project Status Board (`.ai-context/status.md`)**: Updated active spec status and daily log.
   - **5. Prompt History (`.ai-context/prompt_history.md`)**: Appended turn log entry (STRICT APPEND-ONLY RULE).

6. **Continuous Review Loop**:
   - After completing the review for the selected spec, **automatically loop back** to display the updated listing of remaining pending PR reviews assigned to that reviewer!
   - If no more pending reviews remain, display:
     > *"All assigned PR reviews have been completed! Would you like to select an approved spec to start development on?"*

---

## 4. Option 2 / Developer Work Selection & Pre-Development Verification Protocol

1. When starting development on a feature, ask:
   > **"Which approved spec would you like to start development on?"**
2. Display ONLY specs eligible for development (Spec status = `Approved` / Gate 1 Status = `Approved`).
3. If no approved spec exists, display:
   > **"No approved spec is currently available for development. Please complete the PR Gate 1 approval process before proceeding."**
4. **Pre-Development PR Review Check & Developer Notification**:
   Before creating plans, tasks, or code, inspect `.ai-context/pr_reviews/GATE1-<slug>-*.md` for the selected spec and notify the developer:
   - Display Gate 1 Reviewer Name, Email, Review Date, Description, and Comments.
   - **Case A: Gate 1 Status = `Approved`**:
     - Notify developer of approval comments and guide them through:
       1. Create Implementation Plan (`.plan.md`)
       2. Create Tasks Breakdown (`.tasks.md`)
       3. Create Test Cases (`.test_cases.md`)
       4. Execute TDD RED (`tests/`) -> GREEN (`src/`)
       5. Run full test suite to confirm 100% PASS
       6. Submit for Gate 2 Review (`.ai-context/pr_reviews/GATE2-<slug>-*.md`)
   - **Case B: Gate 1 Status = `Rejected` / `Changes Requested`**:
     - **STRICT DEVELOPMENT BLOCK**: Block developer from creating `.plan.md`, `.tasks.md`, `.test_cases.md`, or writing code!
     - Display high-priority rejection warning with reviewer comments.
     - Direct spec author to update `.ai-context/specs/<slug>.spec.md` and re-submit for Gate 1 approval.

---

## 5. Role Separation & Local-Only Handling

- **Reviewer Identity**: Confers official Gate 1 & Gate 2 approval/rejection rights. Pulling code does NOT confer reviewer approval rights unless user is assigned reviewer.
- **Developer Identity**: Confers rights to pull code, select approved specs, write implementation code, and submit for Gate 2 review.
- **Local-Only Scenario**: If project is not pushed to Git, Git credential validation is bypassed, but complete reviewer name, email/User ID, review comments, description, and timestamps MUST be captured in the review record.
