# Spec: Payments & History

## Spec ID
payments

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-015
.ai-context/BRD.md#BRD-016

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Awaiting Gate 1 Spec Peer Review |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
A landlord shall be able to record a payment (Room Rent or Electric) against an outstanding bill and retrieve payment history, satisfying BRD-015 and BRD-016.

## Context
- Builds on: .ai-context/architecture.md (`payments` proposed module — "settles bills from either `shops` (room rent) or `billing` (electric)")
- Related: .ai-context/specs/billing.spec.md — electric bills recorded there are settled here, referenced by bill id only (Microservice Readiness rule: no direct DB joins across module boundaries).
- Related: .ai-context/specs/shops.spec.md — room rent obligations originate from a shop/agreement.
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27)

## API Contract (Mandatory if API surface exists)

### payments.API01 — POST /payments
**Request payload:**
```json
{ "billType": "electric", "billId": "string", "amount": 0, "billingPeriod": "string", "paidOnDate": "string", "method": "cash", "remark": "string", "attachment": "string" }
```

**Success response (`201`):**
```json
{ "paymentId": "string", "billId": "string", "amount": 0, "status": "recorded" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Non-positive amount, or missing required field | `{ "error": "VALIDATION_ERROR" }` |
| 404 | Referenced bill not found, or not owned by caller | `{ "error": "BILL_NOT_FOUND" }` |
| 409 | Bill is already fully paid | `{ "error": "BILL_ALREADY_PAID" }` |

### payments.API02 — GET /payments
**Request payload:** _(none — query params optional, e.g. `?billType=electric`)_

**Success response (`200`):**
```json
{ "payments": [ { "paymentId": "string", "billType": "electric", "amount": 0, "paidOnDate": "string", "method": "cash" } ] }
```

**Exceptions:** _(none beyond standard auth failures)_

### payments.API03 — GET /admin/payments
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "payments": [ { "paymentId": "string", "landlordId": "string", "billType": "electric", "amount": 0 } ] }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |

## Acceptance Criteria

1. payments.AC1 — Given a valid, unpaid bill and a positive amount, when `POST /payments` is submitted, then a payment is recorded, the referenced bill transitions from `Unpaid` to `Paid`, and a receipt-ready payment record is returned, satisfying BRD-015 and R22.
2. payments.AC2 — Given a non-positive or missing amount, when `POST /payments` is submitted, then the request is rejected with `VALIDATION_ERROR` and no payment or bill-status change occurs.
3. payments.AC3 — Given a bill that is already `Paid`, when `POST /payments` is submitted against it again, then the request is rejected with `BILL_ALREADY_PAID`.
4. payments.AC4 — Given a bill owned by another landlord, when a `landlord` caller submits `POST /payments` referencing it, then the request is rejected with `404 BILL_NOT_FOUND`.
5. payments.AC5 — Given an authenticated landlord, when `GET /payments` is called, then only that landlord's own payment history is returned, satisfying BRD-016.
6. payments.AC6 — Given a landlord with no recorded payments, when `GET /payments` is called, then an empty array is returned, not an error (R11).
7. payments.AC7 — Given a `super_admin` caller, when `GET /admin/payments` is called, then payment records across all landlords are returned.
8. payments.AC8 — Given a `landlord`-role caller, when `GET /admin/payments` is called, then the request is rejected with `403 FORBIDDEN`.

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| payments.UT01 | AC1 | Record payment against an unpaid bill | 201, bill transitions to Paid |
| payments.UT02 | AC2 | Submit payment with amount=0 | 400 VALIDATION_ERROR |
| payments.UT03 | AC3 | Submit payment against an already-paid bill | 409 BILL_ALREADY_PAID |
| payments.UT04 | AC4 | Submit payment referencing another landlord's bill | 404 BILL_NOT_FOUND |
| payments.UT05 | AC5 | Landlord lists own payment history | Only own payments returned |
| payments.UT06 | AC6 | Landlord with zero payments lists history | 200, empty array |
| payments.UT07 | AC7 | Super-Admin lists all payments | Cross-landlord results |
| payments.UT08 | AC8 | Landlord-role calls admin payments endpoint | 403 FORBIDDEN |

## Explicitly Out of Scope
- Online payment gateway integration — explicitly Out of Scope in the BRD.
- Partial payments / installment settlement — the BRD describes a single payment settling a bill, not partial balances; partial-payment handling is unresolved.
- Payment confirmation receipt rendering — client-side presentation concern; this spec returns the data the receipt is built from.

## Non-Functional Constraints (from constitution.md)
- Payment amounts and records must be protected in transit and at rest (Security Posture, BRD §31).
- Monetary fields must be positive numeric (Security Posture, BRD §19).
- Every request requires a valid session token, scoped by `landlordId` from the token (Security Posture, R23).
