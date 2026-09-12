# Spec: Billing Configuration

## Spec ID
billing

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-013
.ai-context/BRD.md#BRD-014

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Awaiting Gate 1 Spec Peer Review |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
A landlord shall be able to record an electric bill for a shop, with its amount calculated from a global, landlord-configurable service rate, satisfying BRD-013 and BRD-014.

## Context
- Builds on: .ai-context/architecture.md (`billing` proposed module — "Global Service Rate config + Electric Bill calculation form one cohesive billing concern")
- Related: .ai-context/specs/shops.spec.md — an electric bill is recorded against a shop (`GET /shops/:id/bills`), referenced by shop id only, never via direct DB join across module boundaries (Microservice Readiness rule).
- Related: .ai-context/specs/payments.spec.md — bills recorded here are later settled by a payment; this spec does not itself capture payment.
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27)

## API Contract (Mandatory if API surface exists)

### billing.API01 — GET /shops/:id/bills
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "bills": [ { "billId": "string", "billingPeriod": "string", "amount": 0, "status": "unpaid" } ] }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Shop not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |

### billing.API02 — POST /bills/electric
**Request payload:**
```json
{ "shopId": "string", "billingPeriod": "string", "previousUnit": 0, "currentUnit": 0, "extraAmount": 0, "remark": "string", "attachment": "string" }
```

**Success response (`201`):**
```json
{ "billId": "string", "billingPeriod": "string", "amount": 0, "status": "unpaid" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | `currentUnit` is lower than `previousUnit` | `{ "error": "INVALID_METER_READING" }` |
| 400 | Backdated more than 7 days, or missing required field | `{ "error": "VALIDATION_ERROR" }` |
| 404 | `shopId` not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |

### billing.API03 — GET /service-rate
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "electricRate": 0, "lateRentCharge": 0 }
```

**Exceptions:** _(none beyond standard auth failures)_

### billing.API04 — PUT /service-rate
**Request payload:**
```json
{ "electricRate": 0, "lateRentCharge": 0 }
```

**Success response (`200`):**
```json
{ "electricRate": 0, "lateRentCharge": 0 }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Non-positive or non-numeric rate value | `{ "error": "VALIDATION_ERROR" }` |

## Acceptance Criteria

1. billing.AC1 — Given a shop with a configured electric rate, when `POST /bills/electric` is submitted with valid meter readings, then `amount` is computed as `(currentUnit − previousUnit) × electricRate + extraAmount`, satisfying BRD-013 and R08.
2. billing.AC2 — Given `currentUnit` lower than `previousUnit`, when `POST /bills/electric` is submitted, then the request is rejected with `INVALID_METER_READING`, satisfying R09.
3. billing.AC3 — Given a billing date more than 7 days in the past, when `POST /bills/electric` is submitted, then the request is rejected with `VALIDATION_ERROR`, satisfying R06's backdating limit.
4. billing.AC4 — Given a billing date in the future, when `POST /bills/electric` is submitted, then it is accepted — R06 places no upper limit on future dates.
5. billing.AC5 — Given a landlord submits new `electricRate`/`lateRentCharge` values, when `PUT /service-rate` is called, then the global configuration is updated and subsequent bill calculations use the new rate, satisfying BRD-014.
6. billing.AC6 — Given a non-positive rate value, when `PUT /service-rate` is called, then the request is rejected with `VALIDATION_ERROR`, satisfying the monetary-field validation rule.
7. billing.AC7 — Given a `landlord` caller, when `GET /shops/:id/bills` is called for a shop owned by another landlord, then the request is rejected with `404 SHOP_NOT_FOUND`.
8. billing.AC8 — Given a `super_admin` caller, when `GET /shops/:id/bills` or `GET /service-rate` is called, then it may view across all landlords (Role & Permission Matrix: "Bills / Service Rate: View — all landlords").

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| billing.UT01 | AC1 | Record electric bill with valid readings | 201, amount computed correctly |
| billing.UT02 | AC2 | currentUnit < previousUnit | 400 INVALID_METER_READING |
| billing.UT03 | AC3 | Billing date 10 days in the past | 400 VALIDATION_ERROR |
| billing.UT04 | AC4 | Billing date 30 days in the future | 201, accepted |
| billing.UT05 | AC5 | Update service rate | 200, new rate persisted |
| billing.UT06 | AC6 | Update service rate with negative value | 400 VALIDATION_ERROR |
| billing.UT07 | AC7 | Landlord requests another landlord's shop bills | 404 SHOP_NOT_FOUND |
| billing.UT08 | AC8 | Super-Admin lists bills across landlords | 200, cross-landlord results |

## Explicitly Out of Scope
- Payment capture against a bill — owned by `payments.spec.md`; this spec only creates the bill and computes its amount.
- Landlord/Owner (non-electric) rent billing — the BRD's Business Data Model ties Electric Bill specifically to meter-based calculation; rent itself is defined on the Agreement, not billed through this module.
- Per-shop rate overrides — the BRD specifies a single **global** Service Rate Config, not per-shop rates.

## Non-Functional Constraints (from constitution.md)
- Current meter reading must never be lower than the previous reading — enforced server-side (Non-Functional Baselines, R09).
- Monetary fields (rate, extra amount) must be positive numeric (Security Posture, BRD §19).
- Every request requires a valid session token, scoped by `landlordId` from the token (Security Posture, R23).
