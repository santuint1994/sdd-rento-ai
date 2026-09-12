# Spec: Dashboard

## Spec ID
dashboard

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-005

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Awaiting Gate 1 Spec Peer Review |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
A landlord shall see a read-only portfolio summary aggregating their Shops, Payments, and Agreements on a single dashboard endpoint, satisfying BRD-005; a Super-Admin shall see the same aggregation across all landlords for oversight.

## Context
- Builds on: .ai-context/architecture.md (`dashboard` proposed module — "kept separate so it stays a thin reporting layer, not a place business rules leak into")
- Related: reads only from `shops`, `billing`, and `payments` data; owns no entity of its own and performs no writes.
- Related: .ai-context/specs/access-control.spec.md — role-scoping middleware determines landlord-scoped vs. cross-landlord aggregation.
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27)

## API Contract (Mandatory if API surface exists)

### dashboard.API01 — GET /dashboard/summary
**Request payload:** _(none — identity derived from session token)_

**Success response (`200`):**
```json
{ "activeShops": 0, "totalOutstanding": 0, "collectedThisPeriod": 0, "activeAgreements": 0 }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 401 | No valid session | `{ "error": "UNAUTHENTICATED" }` |

### dashboard.API02 — GET /admin/dashboard/summary
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "activeShops": 0, "totalOutstanding": 0, "collectedThisPeriod": 0, "activeAgreements": 0, "landlordCount": 0 }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 403 | Caller role is not `super_admin` | `{ "error": "FORBIDDEN" }` |

> **Note:** The BRD does not enumerate the exact KPI set shown on the Home screen beyond "summary metrics" (BRD-005). The four metrics above (active shops, outstanding balance, collected-this-period, active agreements) are a reasonable minimum inferred from the linked entities (Shop, Payment, Agreement); confirm the definitive KPI list with the Technical Lead/stakeholder before Gate 2, since this shapes the response contract.

## Acceptance Criteria

1. dashboard.AC1 — Given a `landlord` caller, when `GET /dashboard/summary` is called, then all returned metrics are computed only from that landlord's own shops/payments/agreements.
2. dashboard.AC2 — Given a landlord with no shops or payments yet, when `GET /dashboard/summary` is called, then zeroed metrics are returned with a `200`, not an error (parallels R11's "No Data Found" behavior at the aggregate level).
3. dashboard.AC3 — Given a `super_admin` caller, when `GET /admin/dashboard/summary` is called, then metrics aggregate across every landlord's data.
4. dashboard.AC4 — Given a `landlord`-role caller, when `GET /admin/dashboard/summary` is called, then the request is rejected with `403 FORBIDDEN`.

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| dashboard.UT01 | AC1 | Landlord with existing shops/payments fetches summary | 200, metrics scoped to that landlord |
| dashboard.UT02 | AC2 | Landlord with zero shops fetches summary | 200, all metrics zero |
| dashboard.UT03 | AC3 | Super-Admin fetches admin summary | 200, cross-landlord totals |
| dashboard.UT04 | AC4 | Landlord-role calls admin summary endpoint | 403 FORBIDDEN |

## Explicitly Out of Scope
- Defining the final, confirmed KPI list — pending Technical Lead sign-off (see note above).
- Any write operation — this module is strictly read/aggregation.
- Historical trend charts or time-series data — the BRD describes only a summary, not a history view (a separate, unspecified "History screen" is BRD Open Question #1).

## Non-Functional Constraints (from constitution.md)
- Every request requires a valid session token (Security Posture, BRD §22).
- Aggregation queries must be indexed and optimized appropriately (Non-Functional Baselines, INT baseline) since this endpoint aggregates across multiple tables/modules.
- Backend p95 latency targets are `[Open]` per constitution.md — not yet a binding constraint for this aggregation endpoint.
