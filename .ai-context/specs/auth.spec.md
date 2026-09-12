# Spec: Authentication & Session

## Spec ID
auth

## Status
Approved

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-001
.ai-context/BRD.md#BRD-002
.ai-context/BRD.md#BRD-003

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | 2026-09-13 01:15:15 | APPROVED | auth.spec.md (Authentication & Session) formally approved at Gate 1 Spec Peer Review. See dedicated review record: [GATE1-auth-20260913-011515.md](file:///c:/Users/Supratim_Jetty/Desktop/office%20projects/AI_Projects/sdd-rento-ai/.ai-context/pr_reviews/GATE1-auth-20260913-011515.md) |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
A landlord shall be able to register an account, authenticate with email and password to receive a session (JWT access/refresh token pair), recover a forgotten password via OTP, and log out to invalidate the local session — establishing the identity and session foundation every other protected module (`account`, `dashboard`, `shops`, `rooms`, `billing`, `payments`, `access-control`) depends on.

## Context
- Builds on: .ai-context/architecture.md (`auth` proposed module; Authentication & Security section)
- Related: .ai-context/specs/access-control.spec.md (not yet authored) — the `role` and `landlordId` claims this spec's tokens carry are consumed by `access-control`'s authorization middleware for every other module.
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27)

## API Contract (Mandatory if API surface exists)

### auth.API01 — POST /auth/signup
**Request payload:**
```json
{ "email": "string", "password": "string", "name": "string" }
```

**Success response (`201`):**
```json
{ "landlordId": "string", "email": "string", "role": "landlord" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Missing/invalid required field | `{ "error": "VALIDATION_ERROR", "fields": ["email"] }` |
| 409 | Email already registered | `{ "error": "EMAIL_ALREADY_EXISTS" }` |

### auth.API02 — POST /auth/login
**Request payload:**
```json
{ "email": "string", "password": "string" }
```

**Success response (`200`):**
```json
{ "accessToken": "string", "refreshToken": "string", "role": "landlord", "landlordId": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 401 | Email not registered, or password does not match | `{ "error": "INVALID_CREDENTIALS" }` |

### auth.API03 — POST /auth/otp/send
**Request payload:**
```json
{ "email": "string" }
```

**Success response (`200`):**
```json
{ "sent": true, "resendAfterSeconds": 75 }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Email not registered | `{ "error": "ACCOUNT_NOT_FOUND" }` |
| 429 | Resend requested before cooldown elapsed | `{ "error": "OTP_RESEND_COOLDOWN", "retryAfterSeconds": 42 }` |

### auth.API04 — POST /auth/otp/verify
**Request payload:**
```json
{ "email": "string", "otp": "string" }
```

**Success response (`200`):**
```json
{ "verified": true, "resetToken": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | OTP does not match the one issued, or has expired | `{ "error": "INVALID_OTP" }` |

### auth.API05 — POST /auth/password/reset
**Request payload:**
```json
{ "resetToken": "string", "newPassword": "string" }
```

**Success response (`200`):**
```json
{ "reset": true }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Reset token invalid, expired, or already used | `{ "error": "INVALID_RESET_TOKEN" }` |

### auth.API06 — POST /auth/logout
**Request payload:**
```json
{ "refreshToken": "string" }
```

**Success response (`200`):**
```json
{ "loggedOut": true }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 401 | No valid session on the request | `{ "error": "NO_ACTIVE_SESSION" }` |

## Acceptance Criteria

1. auth.AC1 — Given an unregistered email, when a landlord submits Sign Up with a valid email/password/name, then a new `landlord`-role account is created and BRD-002 is satisfied. (R20)
2. auth.AC2 — Given an already-registered email, when Sign Up is submitted again with that email, then the request is rejected with `EMAIL_ALREADY_EXISTS` and no duplicate account is created.
3. auth.AC3 — Given a registered email and its correct password, when Login is submitted, then an access/refresh token pair is issued embedding `role` and `landlordId`, satisfying BRD-001 and R17.
4. auth.AC4 — Given a registered email with an incorrect password, or an unregistered email, when Login is submitted, then the request is rejected with `INVALID_CREDENTIALS` without revealing which field was wrong.
5. auth.AC5 — Given a registered email, when OTP is requested, then a one-time code is generated and a resend is blocked for 75 seconds, satisfying R19.
6. auth.AC6 — Given an OTP requested less than 75 seconds ago, when OTP is requested again, then the request is rejected with `OTP_RESEND_COOLDOWN`.
7. auth.AC7 — Given a valid, unexpired OTP for an email, when it is verified, then a short-lived reset token is issued and the password reset endpoint accepts it, satisfying BRD-003.
8. auth.AC8 — Given an invalid or expired OTP, when it is verified, then the request is rejected with `INVALID_OTP` and no reset token is issued.
9. auth.AC9 — Given a valid reset token, when a new password is submitted, then the account's password is updated and the reset token cannot be reused.
10. auth.AC10 — Given an active session, when Logout is called, then the refresh token is invalidated server-side, satisfying R18.
11. auth.AC11 — Given an expired or invalid access token, when any protected endpoint is called, then the request is rejected with `401` prompting re-authentication, satisfying Session Expiry (BRD §22).

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| auth.UT01 | AC1 | Sign up with new email | 201, landlord created with role=landlord |
| auth.UT02 | AC2 | Sign up with existing email | 409 EMAIL_ALREADY_EXISTS |
| auth.UT03 | AC3 | Login with correct credentials | 200, tokens contain role & landlordId |
| auth.UT04 | AC4 | Login with wrong password | 401 INVALID_CREDENTIALS |
| auth.UT05 | AC4 | Login with unregistered email | 401 INVALID_CREDENTIALS |
| auth.UT06 | AC5 | Request OTP for registered email | 200, resendAfterSeconds=75 |
| auth.UT07 | AC6 | Request OTP twice within 75s | 429 OTP_RESEND_COOLDOWN |
| auth.UT08 | AC7 | Verify correct, unexpired OTP | 200, resetToken issued |
| auth.UT09 | AC8 | Verify incorrect OTP | 400 INVALID_OTP |
| auth.UT10 | AC9 | Reset password with valid resetToken | 200, password updated |
| auth.UT11 | AC9 | Reuse a spent resetToken | 400 INVALID_RESET_TOKEN |
| auth.UT12 | AC10 | Logout with active refresh token | 200, refresh token invalidated |
| auth.UT13 | AC11 | Call a protected route with expired access token | 401 |

## Explicitly Out of Scope
- Role assignment and Super-Admin account administration — owned by the `access-control` module (BRD-017), not this spec.
- Profile field management after account creation — owned by the `account` module (BRD-004).
- Mobile client OTP delivery channel/UI (SMS/email presentation) — client-side, out of this backend-only repository's scope.
- Multi-factor authentication beyond OTP-based password recovery — not specified in the BRD.

## Non-Functional Constraints (from constitution.md)
- Session tokens: JWT access/refresh strategy, stored securely by the client; this spec only issues/validates them (Security Posture).
- Every protected route beyond Login/Sign Up/Forgot Password/OTP requires a valid session token (Security Posture, BRD §22).
- No hardcoded secrets — JWT signing keys read from environment variables only (Security Posture).
- Contact fields (email) must be format-validated (Security Posture, BRD §19).
- Backend p95 latency/availability targets are `[Open]` per constitution.md — not a binding constraint yet.
