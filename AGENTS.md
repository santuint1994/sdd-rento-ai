# AGENTS.md — INT AI-First Engineering Policy

This file governs how any AI coding assistant (Claude, Gemini, Cursor, Windsurf, Copilot, or otherwise) must operate inside this repository. It is vendor-agnostic and repository-local: no external or cloud-only AI configuration is required to understand or enforce these rules.

## 1. Authority Hierarchy

When guidance conflicts, resolve in this order (highest authority first):

1. **`.ai-context/constitution.md`** — project-specific constraints (testing, security, architecture, non-functional baselines, versioning). Derived from the project BRD when available.
2. **`.agent/rules/`** — organization-wide INT engineering standards and governance rules (content-equivalent copy of the INT Control Plane; must not be edited locally).
3. **`.agent/workflows/`** — organization-wide INT process workflows (spec lifecycle, PR gates, incident/hotfix/release management).
4. **`.agents/skills/`** — project-local copies of the INT SDD skills that operationalize the workflows above.
5. **This file (`AGENTS.md`)** — repository-wide policy summary and pointer to the above.

A BRD-provided constitution constraint always takes precedence over a generic INT rule. If a BRD constitution conflicts with an organization-level rule in `.agent/rules/`, the conflict MUST be flagged for human review — it must never be silently resolved by an AI assistant.

## 2. Source of Truth Boundaries

- `.agent/` — the INT Control Plane. Organization-wide, vendor-neutral engineering standards and workflows. Treat as **read-only**; it is dynamically synced from the org-level source and must remain content-equivalent.
- `.ai-context/` — this project's living knowledge base: constitution, project context, architecture, BRD, specs, plans, tasks, test cases, PR reviews, decisions (ADRs), incidents, hotfixes, releases, change requests, and prompt history.
- `.agents/skills/` — project-local, portable copies of the INT SDD skills so any assistant working in this repo (not just one tied to a specific vendor) can execute the same lifecycle.
- `src/`, `tests/`, `docs/` — the execution layer (application code, automated tests, and supplementary documentation).

## 3. Development Lifecycle (Spec-Driven Development)

Every feature or change follows this lifecycle, tracked in `.ai-context/status.md`:

```
Draft → Gate 1 Peer Review (Spec) → Approved → Plan → Tasks →
Test Cases (TDD RED) → Implementation (TDD GREEN) → Gate 2 Code Review →
Ready for Release → Released (vX.Y.Z)
```

- **Gate 1 (Spec Review):** validates requirement completeness, scope, business rules, edge cases, and acceptance criteria before any planning or code is written.
- **Gate 2 (Code Review):** validates implementation against the approved spec, code quality, security, test coverage, and regression impact before release.
- A spec **Rejected** or marked **Changes Requested** at Gate 1 blocks all downstream planning, task generation, test drafting, and implementation until the spec is revised and re-approved.
- Production incidents are triaged and routed to a Spec Gap, an Implementation Defect (hotfix), or a New Requirement (BRD ingestion) — see `.agents/skills/int-incident-management/`.
- Hotfixes follow a compressed, test-first path with a **mandatory post-hoc Gate 2 review** — see `.agents/skills/int-hotfix-management/`.
- Releases are assembled from specs that have passed both gates — see `.agents/skills/int-release-management/`.

## 4. Core Governance Rules

- Never invent technology choices, reviewer assignments, or architectural decisions without explicit user confirmation or BRD backing.
- Never modify files under `.agent/` — it is the authoritative, dynamically-synced organizational Control Plane.
- Never write absolute local filesystem paths into any repository artifact; all cross-references must be relative to the repository root.
- Never include comments or commit messages indicating that code was AI-generated.
- Artifacts under `.ai-context/specs/`, `plans/`, `tasks/`, and `test_cases/` are flat files (`<feature-slug>.<type>.md`) — never nested in per-feature subdirectories.
- `.ai-context/prompt_history.md` is strictly append-only; never overwrite or delete prior entries.
- Reviewer approval requires identity match: the person approving a Gate 1 or Gate 2 review must match the assigned reviewer on the spec.

## 5. Where To Look

| Need | Location |
|---|---|
| Org-wide coding/security/PR-gate standards | `.agent/rules/` |
| Org-wide process workflows | `.agent/workflows/` |
| This project's constraints | `.ai-context/constitution.md` |
| This project's tech stack & architecture | `.ai-context/project_context.md`, `.ai-context/architecture.md` |
| Current status of all specs | `.ai-context/status.md` |
| How to run the SDD lifecycle | `.agents/skills/int-sdd-lifecycle/SKILL.md` |
| How to ingest a BRD | `.agents/skills/int-brd-ingestion/SKILL.md` |
| How to handle a production incident | `.agents/skills/int-incident-management/SKILL.md` |
| How to ship a hotfix | `.agents/skills/int-hotfix-management/SKILL.md` |
| How to cut a release | `.agents/skills/int-release-management/SKILL.md` |
| How to resume work after a session restart | `.agents/skills/int-session-continuation/SKILL.md` |
