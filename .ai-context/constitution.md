# Project Constitution

> The ingested BRD (`docs/Rento BRD.pdf`) does not contain a dedicated Constitution / Engineering Constitution section. This file therefore combines BRD-sourced constraints (Security Requirements §31, Non-Functional Requirements §30) with the INT baseline for anything the BRD leaves unspecified. BRD-sourced lines are marked `[BRD]`; unspecified areas are marked `[Open]` and require Technical Lead confirmation before they become binding.

## Testing Discipline
- `[Open]` The BRD does not specify backend test coverage floors or TDD mandates. INT baseline applies: test-first (TDD) discipline — test cases drafted from the approved spec's Acceptance Criteria before implementation (TDD RED), then implementation proceeds until tests pass (TDD GREEN).
- Automated tests live under `tests/backend/`, mirroring the `modules/`, `config/`, and `shared/` structure of `src/backend/`.
- `.ai-context/test_cases/` holds test-case specifications and acceptance-oriented scenarios only — not executable test code.

## Security Posture
- `[BRD §31]` Session tokens are stored in secure, encrypted storage; the confirmed strategy is JWT (access/refresh tokens) (`.ai-context/project_context.md`).
- `[BRD §31]` Tenant PII (names, addresses, phone numbers, identity documents) must be transmitted and stored **encrypted**.
- `[BRD §31]` Payment amounts and payment records must be protected **in transit and at rest**.
- `[BRD §31]` Every API request beyond authentication requires a valid session token (Section 22: Protected routes).
- `[BRD §19]` Contact fields (email, phone, alternate phone) must be format-validated; phone numbers are digit-only and length-bound. Monetary fields must be positive numeric.
- Never hardcode secrets, API keys, or database credentials — read from environment variables (`process.env`), never committed (`.env` is git-ignored).
- Avoid `eval()` or execution of arbitrary code.

## Architectural Constraints
- Architecture style: **Monolithic** (single deployable backend service) — confirmed at project setup.
- Backend technology: Node.js + Express.
- Database: PostgreSQL, accessed via Sequelize ORM.
- `[BRD §21]` Core entities: Landlord, Shop, Room, Tenant, Agreement, Electric Bill, Payment, Service Rate Config. See `.ai-context/BRD.md` § Business Data Model.
- `[BRD §27]` The backend must expose the API surface listed in `.ai-context/BRD.md` § Backend API Contract.
- No new datastore or major architectural shift may be introduced without a corresponding ADR under `.ai-context/decisions/`.

## Non-Functional Baselines
- `[BRD §8, R09]` Electric bill current meter reading must never be lower than the previous reading — enforce server-side, not just client-side.
- `[BRD §18, R07]` An agreement's end date must fall after its start date; validate server-side even though the client also selects dates independently.
- `[Open]` The BRD does not specify backend p95 latency, throughput, or availability targets. Do not invent numeric SLAs — confirm with the Technical Lead and record the decision as an ADR before treating any specific figure as binding.
- `[Open]` Recovery objectives (RPO/RTO) are not specified in the BRD — same rule applies.
- Prevent event-loop blocking; offload heavy synchronous computation to worker threads or external services (INT baseline).
- Database queries must be indexed and optimized appropriately (INT baseline).
- Deployment target: Unknown / TBD — to be confirmed before first release.

## Versioning Rules
- Releases are tracked as `RELEASE-vX.Y.Z.md` under `.ai-context/releases/`.
- A spec may only be marked `Released (vX.Y.Z)` after passing both Gate 1 and Gate 2 reviews.
- `[BRD]` No API versioning scheme (e.g. `/api/v1`) is specified in the BRD — confirm with the Technical Lead before the first `spec.md` defines route paths, and record the decision as an ADR if it introduces a constraint.
