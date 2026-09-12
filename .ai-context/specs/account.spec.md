# Spec: Account & Settings

## Spec ID
account

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-004

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Awaiting Gate 1 Spec Peer Review |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
An authenticated account holder (`landlord` or `super_admin`) shall be able to view and update their own profile, satisfying BRD-004, with all reads/writes scoped to the caller's own account only.

## Context
- Builds on: .ai-context/architecture.md (`account` proposed module)
- Related: .ai-context/specs/auth.spec.md — profile access requires a valid session issued by `auth`; password changes are handled by `auth`'s OTP/reset flow, not this spec.
- Related: .ai-context/specs/access-control.spec.md — role-scoping middleware restricts this module's endpoints to the caller's own record regardless of role.
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27)

## API Contract (Mandatory if API surface exists)

### account.API01 — GET /profile
**Request payload:** _(none — identity derived from session token)_

**Success response (`200`):**
```json
{ "landlordId": "string", "name": "string", "email": "string", "phone": "string", "role": "landlord" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 401 | No valid session | `{ "error": "UNAUTHENTICATED" }` |

### account.API02 — PUT /profile
**Request payload:**
```json
{ "name": "string", "phone": "string" }
```

**Success response (`200`):**
```json
{ "landlordId": "string", "name": "string", "email": "string", "phone": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Invalid phone format, or empty required field | `{ "error": "VALIDATION_ERROR", "fields": ["phone"] }` |
| 401 | No valid session | `{ "error": "UNAUTHENTICATED" }` |

> **Note:** The BRD's "Edit Profile" screen (BRD-004) does not enumerate its exact field set beyond referencing landlord contact/profile data. `name` and `phone` are the fields carried over from the BRD's Business Data Model note on Landlord "profile fields"; `email` is treated as read-only here since it is also the login identifier owned by `auth`. Confirm the complete editable field list with the Technical Lead before Gate 2.

## Acceptance Criteria

1. account.AC1 — Given an authenticated caller, when `GET /profile` is called, then only that caller's own profile is returned, never another account's.
2. account.AC2 — Given an authenticated caller submits a valid name/phone update, when `PUT /profile` is called, then the changes are persisted and returned.
3. account.AC3 — Given an invalid phone format, when `PUT /profile` is called, then the request is rejected with `VALIDATION_ERROR` and no partial update occurs.
4. account.AC4 — Given a `super_admin` caller, when `GET`/`PUT /profile` is called, then it affects only the Super-Admin's own account — it is not a route to editing any landlord's profile (that is `access-control`'s concern).
5. account.AC5 — Given no session token, when either endpoint is called, then the request is rejected with `401 UNAUTHENTICATED`.

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| account.UT01 | AC1 | Authenticated landlord fetches profile | 200, own profile only |
| account.UT02 | AC2 | Update name and phone with valid data | 200, fields persisted |
| account.UT03 | AC3 | Update with malformed phone | 400 VALIDATION_ERROR |
| account.UT04 | AC4 | Super-Admin fetches/updates own profile | 200, scoped to admin's own record |
| account.UT05 | AC5 | No token supplied | 401 UNAUTHENTICATED |

## Explicitly Out of Scope
- Password change / reset — owned by `auth.spec.md` (OTP-based reset flow).
- Editing any account other than the caller's own — landlord-account administration is owned by `access-control.spec.md`.
- Account deletion — not specified in the BRD.

## Non-Functional Constraints (from constitution.md)
- Contact fields (phone) must be format-validated, digit-only and length-bound (Security Posture, BRD §19).
- Every request requires a valid session token (Security Posture, BRD §22).
- Profile identity (`landlordId`) is derived from the token, never a request parameter (Security Posture, R23).
