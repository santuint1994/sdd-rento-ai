# Spec: Access Control & Role Management

## Spec ID
access-control

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-017

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Awaiting Gate 1 Spec Peer Review |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
A `super_admin` shall be able to create, view, edit, activate/deactivate landlord accounts and assign/revoke roles, and every protected endpoint in every other module shall enforce that a `landlord` caller is restricted to their own records while a `super_admin` caller is unrestricted — satisfying BRD-017 and R23 (added 2026-09-13).

## Context
- Builds on: .ai-context/architecture.md (`access-control` proposed module; Authorization section)
- Related: .ai-context/specs/auth.spec.md — consumes the `role` and `landlordId` claims issued at login; this spec does not reissue or validate tokens itself, only authorizes based on their claims.
- Related: every other module spec (`account`, `dashboard`, `shops`, `rooms`, `billing`, `payments`) applies this module's role-scoping middleware locally per route, per the architecture's cross-module rule (no pre-filtered trust).
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27), § Role & Permission Matrix

## API Contract (Mandatory if API surface exists)

### access-control.API01 — GET /admin/landlords
**Request payload:** _(none — query params optional, e.g. `?status=active`)_

**Success response (`200`):**
```json
{ "landlords": [ { "landlordId": "string", "email": "string", "role": "landlord", "status": "active" } ] }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |

### access-control.API02 — POST /admin/landlords
**Request payload:**
```json
{ "email": "string", "password": "string", "name": "string", "role": "landlord" }
```

**Success response (`201`):**
```json
{ "landlordId": "string", "email": "string", "role": "landlord", "status": "active" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |
| 409 | Email already registered | `{ "error": "EMAIL_ALREADY_EXISTS" }` |

### access-control.API03 — GET /admin/landlords/:id
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "landlordId": "string", "email": "string", "name": "string", "role": "landlord", "status": "active" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |
| 404 | No landlord with that id | `{ "error": "LANDLORD_NOT_FOUND" }` |

### access-control.API04 — PUT /admin/landlords/:id
**Request payload:**
```json
{ "name": "string", "email": "string" }
```

**Success response (`200`):**
```json
{ "landlordId": "string", "email": "string", "name": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |
| 404 | No landlord with that id | `{ "error": "LANDLORD_NOT_FOUND" }` |

### access-control.API05 — PUT /admin/landlords/:id/status
**Request payload:**
```json
{ "status": "active" }
```

**Success response (`200`):**
```json
{ "landlordId": "string", "status": "inactive" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |
| 404 | No landlord with that id | `{ "error": "LANDLORD_NOT_FOUND" }` |

### access-control.API06 — PUT /admin/landlords/:id/role
**Request payload:**
```json
{ "role": "super_admin" }
```

**Success response (`200`):**
```json
{ "landlordId": "string", "role": "super_admin" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Role is not one of the two fixed roles | `{ "error": "INVALID_ROLE" }` |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |
| 404 | No landlord with that id | `{ "error": "LANDLORD_NOT_FOUND" }` |

## Acceptance Criteria

1. access-control.AC1 — Given a `super_admin` caller, when `GET /admin/landlords` is called, then every landlord account across the platform is returned, satisfying BRD-017's cross-landlord oversight.
2. access-control.AC2 — Given a `landlord`-role caller, when any `/admin/*` endpoint is called, then the request is rejected with `403 FORBIDDEN` — no landlord-role token may reach admin functions.
3. access-control.AC3 — Given a `super_admin` caller, when `POST /admin/landlords` is submitted with a new email, then a landlord account is created with the requested role.
4. access-control.AC4 — Given a `super_admin` caller, when `PUT /admin/landlords/:id/status` sets `status: inactive`, then that landlord's account is deactivated and subsequent logins for that account are rejected (enforced by `auth`).
5. access-control.AC5 — Given a `super_admin` caller, when `PUT /admin/landlords/:id/role` assigns or revokes a role, then the account's `role` claim changes and is reflected in that landlord's next-issued token.
6. access-control.AC6 — Given any protected endpoint in `shops`, `rooms`, `billing`, or `payments`, when called with a `landlord`-role token, then all reads/writes are scoped to `WHERE landlordId = :callerId` derived from the token — never from a request parameter or body, satisfying R23.
7. access-control.AC7 — Given any protected endpoint in `shops`, `rooms`, `billing`, or `payments`, when called with a `super_admin`-role token, then the endpoint is exempt from landlord-scoping and may act across all landlords, satisfying BRD-017's oversight requirement.
8. access-control.AC8 — Given a request that supplies a `landlordId` in its body/params that differs from the caller's own token-derived id, when the caller role is `landlord`, then the supplied id is ignored/rejected in favor of the token-derived id (defense against scope-bypass).

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| access-control.UT01 | AC1 | super_admin lists all landlords | 200, full cross-landlord list |
| access-control.UT02 | AC2 | landlord-role calls GET /admin/landlords | 403 FORBIDDEN |
| access-control.UT03 | AC3 | super_admin creates a landlord account | 201, account created |
| access-control.UT04 | AC4 | super_admin deactivates a landlord | 200, status=inactive; later login for that account fails |
| access-control.UT05 | AC5 | super_admin assigns super_admin role to a landlord | 200, role updated |
| access-control.UT06 | AC5 | invalid role value submitted | 400 INVALID_ROLE |
| access-control.UT07 | AC6 | landlord-role token calls GET /shops | results contain only that landlord's shops |
| access-control.UT08 | AC7 | super_admin token calls GET /admin/shops | results span multiple landlords |
| access-control.UT09 | AC8 | landlord token submits a foreign landlordId in payload | request scoped to caller's own id, foreign id ignored |

## Explicitly Out of Scope
- Landlord-created staff/sub-accounts with configurable permissions — explicitly out of scope per the BRD amendment (only the two fixed roles exist).
- Audit logging of Super-Admin actions — BRD Open Question #10, not specified; deferred pending Technical Lead confirmation.
- A Super-Admin-facing console/UI — BRD Open Question #9; this spec is API/backend-only.
- Token issuance/validation itself — owned by `auth.spec.md`.

## Non-Functional Constraints (from constitution.md)
- `landlordId` for scoping must be derived from the authenticated token, never from a client-supplied parameter or body field (Security Posture, BRD §22 R23).
- Every `/admin/*` endpoint requires a valid session token plus `super_admin` role (Security Posture).
- No hardcoded secrets or credentials (Security Posture, INT baseline).
