# Spec: Shop, Tenant & Agreement Management

## Spec ID
shops

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-006
.ai-context/BRD.md#BRD-007
.ai-context/BRD.md#BRD-008
.ai-context/BRD.md#BRD-009
.ai-context/BRD.md#BRD-011
.ai-context/BRD.md#BRD-012

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Awaiting Gate 1 Spec Peer Review |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
A landlord shall be able to onboard a shop together with its tenant and initial agreement in a single guided flow, view/edit/search/close shops, look up tenants (created only via onboarding), and view/create agreements scoped to a shop — satisfying BRD-006 through BRD-009, BRD-011, and BRD-012 as one cohesive module per the architecture's grouping rationale (Tenant is only ever created through shop onboarding; Agreement is always scoped to a shop).

## Context
- Builds on: .ai-context/architecture.md (`shops` proposed module — absorbs Shop Management, Tenant Management, Agreement Management as sub-resources of one onboarding transaction)
- Related: .ai-context/specs/billing.spec.md and .ai-context/specs/payments.spec.md — reference shops by id only, never via direct DB join.
- Related: .ai-context/specs/access-control.spec.md — role-scoping middleware applied per route.
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27)

## API Contract (Mandatory if API surface exists)

### shops.API01 — POST /shops
**Request payload:**
```json
{
  "shop": { "name": "string", "email": "string", "shopType": "string", "description": "string", "medium": "string", "logo": "string", "rentPerMonth": 0, "category": "string" },
  "tenant": { "name": "string", "fatherName": "string", "address": "string", "email": "string", "phone": "string", "alternatePhone": "string", "documents": ["string"] },
  "agreement": { "startDate": "string", "endDate": "string", "rentPerMonth": 0, "payPeriod": "string", "securityDeposit": 0, "lateCharges": 0, "electricalFlag": true }
}
```

**Success response (`201`):**
```json
{ "shopId": "string", "tenantId": "string", "agreementId": "string", "status": "active" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Missing/invalid required field in shop, tenant, or agreement block | `{ "error": "VALIDATION_ERROR", "fields": ["agreement.endDate"] }` |
| 400 | Agreement `endDate` not after `startDate` | `{ "error": "INVALID_AGREEMENT_DATES" }` |

### shops.API02 — GET /shops
**Request payload:** _(none — query params: `?q=<name or owner>`)_

**Success response (`200`):**
```json
{ "shops": [ { "shopId": "string", "name": "string", "category": "string", "status": "active" } ] }
```

**Exceptions:** _(none beyond standard auth failures)_

### shops.API03 — GET /shops/:id
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "shopId": "string", "name": "string", "email": "string", "shopType": "string", "rentPerMonth": 0, "status": "active", "tenant": { "tenantId": "string", "name": "string", "status": "active" } }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Shop not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |

### shops.API04 — PUT /shops/:id
**Request payload:**
```json
{ "name": "string", "email": "string", "shopType": "string", "description": "string", "rentPerMonth": 0, "category": "string" }
```

**Success response (`200`):**
```json
{ "shopId": "string", "name": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Invalid/empty required field | `{ "error": "VALIDATION_ERROR" }` |
| 404 | Shop not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |
| 409 | Shop status is `Closed` | `{ "error": "SHOP_CLOSED" }` |

### shops.API05 — POST /shops/:id/close
**Request payload:**
```json
{ "closeDate": "string", "depositReturned": 0, "remark": "string", "attachment": "string" }
```

**Success response (`200`):**
```json
{ "shopId": "string", "status": "closed", "closeDate": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Shop not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |
| 409 | Shop already `Closed` | `{ "error": "SHOP_ALREADY_CLOSED" }` |

### shops.API06 — DELETE /shops/:id
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "deleted": true }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Shop not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |

### shops.API07 — GET /tenants
**Request payload:** _(none — query params: `?status=active|inactive`)_

**Success response (`200`):**
```json
{ "tenants": [ { "tenantId": "string", "name": "string", "status": "active", "shopId": "string" } ] }
```

**Exceptions:** _(none beyond standard auth failures)_

### shops.API08 — GET /tenants/:id
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "tenantId": "string", "name": "string", "fatherName": "string", "address": "string", "phone": "string", "status": "active", "shopId": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Tenant not found, or not owned by caller | `{ "error": "TENANT_NOT_FOUND" }` |

### shops.API09 — GET /shops/:id/agreements
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "agreements": [ { "agreementId": "string", "startDate": "string", "endDate": "string", "status": "active" } ] }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Shop not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |

### shops.API10 — POST /agreements
**Request payload:**
```json
{ "shopId": "string", "startDate": "string", "endDate": "string", "rentPerMonth": 0, "payPeriod": "string", "securityDeposit": 0, "lateCharges": 0, "electricalFlag": true }
```

**Success response (`201`):**
```json
{ "agreementId": "string", "shopId": "string", "status": "active" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | `endDate` not after `startDate` | `{ "error": "INVALID_AGREEMENT_DATES" }` |
| 400 | Start date backdated more than 7 days | `{ "error": "VALIDATION_ERROR" }` |
| 404 | `shopId` not found, or not owned by caller | `{ "error": "SHOP_NOT_FOUND" }` |

## Acceptance Criteria

1. shops.AC1 — Given valid shop, tenant, and agreement data, when `POST /shops` is submitted, then a Shop, its Tenant, and its initial Agreement are created together in one transaction, satisfying BRD-006.
2. shops.AC2 — Given an agreement block where `endDate` is not after `startDate`, when `POST /shops` or `POST /agreements` is submitted, then the request is rejected with `INVALID_AGREEMENT_DATES` and nothing is persisted, satisfying R07.
3. shops.AC3 — Given an agreement `startDate` more than 7 days in the past, when `POST /shops` or `POST /agreements` is submitted, then the request is rejected, satisfying R06's backdating limit; a future `startDate` has no upper limit and is accepted.
4. shops.AC4 — Given a search query matching a shop's name or owner, when `GET /shops?q=<term>` is called, then only matching shops are returned, satisfying BRD-009 and R12.
5. shops.AC5 — Given an existing shop owned by the caller, when `GET /shops/:id` is called, then shop details plus its current tenant are returned, satisfying BRD-007's shop detail view.
6. shops.AC6 — Given an existing, non-closed shop, when `PUT /shops/:id` is submitted with valid fields, then the shop is updated.
7. shops.AC7 — Given a shop already `Closed`, when `PUT /shops/:id` is submitted, then the request is rejected with `SHOP_CLOSED` — closed shops are not further edited.
8. shops.AC8 — Given an active or expired shop, when `POST /shops/:id/close` is submitted with a close date, deposit-return amount, and remark, then the shop transitions to `Closed` and its closure fields become retrievable, satisfying BRD-008 and R21.
9. shops.AC9 — Given a shop already `Closed`, when `POST /shops/:id/close` is submitted again, then the request is rejected with `SHOP_ALREADY_CLOSED`.
10. shops.AC10 — Given an existing shop, when `DELETE /shops/:id` is called, then only that specific shop record is removed (R15).
11. shops.AC11 — Given `?status=active` or `?status=inactive`, when `GET /tenants` is called, then only tenants matching that status are returned, satisfying R13.
12. shops.AC12 — Given no direct tenant-creation endpoint is exposed, when a client attempts to create a tenant outside of `POST /shops` onboarding, then no such endpoint exists — tenants are only ever created via shop onboarding, satisfying the architecture's coupling rationale.
13. shops.AC13 — Given a shop owned by the caller, when `GET /shops/:id/agreements` is called, then that shop's full agreement history is returned, satisfying BRD-012.
14. shops.AC14 — Given a shop not owned by the caller, when any `/shops/:id/*`, `/tenants/:id`, or agreement endpoint referencing it is called, then the request is rejected with `404` (not `403`).
15. shops.AC15 — Given a `super_admin` caller, when any endpoint in this module is called, then it is not scoped to a single landlord — cross-landlord access is granted per the Role & Permission Matrix (Super-Admin: view-only for Tenants/Agreements, full CRUD for Shops).

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| shops.UT01 | AC1 | Onboard shop+tenant+agreement with valid data | 201, all three records created |
| shops.UT02 | AC2 | Agreement endDate before startDate | 400 INVALID_AGREEMENT_DATES, nothing persisted |
| shops.UT03 | AC3 | Agreement startDate 10 days in the past | 400 VALIDATION_ERROR |
| shops.UT04 | AC3 | Agreement startDate 60 days in the future | 201, accepted |
| shops.UT05 | AC4 | Search shops by owner name substring | Only matching shops returned |
| shops.UT06 | AC5 | Fetch shop detail with tenant | 200, includes tenant sub-object |
| shops.UT07 | AC6 | Update an active shop | 200, fields updated |
| shops.UT08 | AC7 | Update a closed shop | 409 SHOP_CLOSED |
| shops.UT09 | AC8 | Close an active shop with settlement data | 200, status=closed, closeDate set |
| shops.UT10 | AC9 | Close an already-closed shop | 409 SHOP_ALREADY_CLOSED |
| shops.UT11 | AC10 | Delete an existing shop | 200 deleted=true |
| shops.UT12 | AC11 | List tenants filtered by status=inactive | Only inactive tenants returned |
| shops.UT13 | AC13 | List agreements for a shop | Full history returned |
| shops.UT14 | AC14 | Access another landlord's shop by id | 404 SHOP_NOT_FOUND |
| shops.UT15 | AC15 | Super-Admin lists shops across landlords | Cross-landlord results |

## Explicitly Out of Scope
- Whether two active agreements can co-exist for one shop (overlap checking) — BRD Open Question #4, unresolved; this spec does not enforce a non-overlap constraint pending Technical Lead confirmation.
- Whether Shop and Room are the same underlying entity — BRD Open Question #3; treated as separate per architecture.md's proposal.
- Soft-delete / recoverability of a deleted or closed shop — BRD Open Question #6, unresolved.
- Direct tenant creation or editing outside of shop onboarding — tenants are read-only after creation per the Role & Permission Matrix (Landlord: "View; Create via shop onboarding — own only").

## Non-Functional Constraints (from constitution.md)
- An agreement's end date must fall after its start date — enforced server-side (Non-Functional Baselines, R07).
- Tenant PII (name, address, phone, documents) must be transmitted and stored encrypted (Security Posture, BRD §31).
- Every request requires a valid session token, scoped by `landlordId` from the token (Security Posture, R23).
- Monetary fields (rent, deposit, late charges) must be positive numeric (Security Posture, BRD §19).
