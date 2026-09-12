# Spec: Room Management

## Spec ID
rooms

## Status
In Peer Review

## Roles & Assignments
- **Developer:** Unassigned
- **Gate 1 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)
- **Gate 2 Reviewer(s):** Supratim Jetty (supratim.jetty@intglobal.com)

## Linked BRD
.ai-context/BRD.md#BRD-010

## Gate Approvals & History
| Gate | Approver Name | Approver Email/ID | Date/Time | Outcome | Approval Comment / Summary |
|---|---|---|---|---|---|
| Gate 1 (Spec Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Awaiting Gate 1 Spec Peer Review |
| Gate 2 (Code Review) | Supratim Jetty | supratim.jetty@intglobal.com | — | Pending | Not started |

## Intent
A landlord shall be able to create, view, edit, filter, and delete Room records independently of the Shop domain, satisfying BRD-010, with each room uniquely numbered per property and scoped to its owning landlord.

## Context
- Builds on: .ai-context/architecture.md (`rooms` proposed module — "Independent CRUD lifecycle, no dependency on Shop")
- Related: .ai-context/specs/access-control.spec.md — role-scoping middleware applied per route.
- API contract: .ai-context/BRD.md § Backend API Contract (Section 27)

## API Contract (Mandatory if API surface exists)

### rooms.API01 — GET /rooms
**Request payload:** _(none — query params: `?occupancy=occupied|vacant`)_

**Success response (`200`):**
```json
{ "rooms": [ { "roomId": "string", "roomNumber": "string", "size": "string", "occupancy": "vacant" } ] }
```

**Exceptions:** _(none beyond standard auth failures)_

### rooms.API02 — POST /rooms
**Request payload:**
```json
{ "roomNumber": "string", "size": "string", "company": "string", "owner": "string", "photo": "string", "attachment": "string" }
```

**Success response (`201`):**
```json
{ "roomId": "string", "roomNumber": "string", "occupancy": "vacant" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Missing required field | `{ "error": "VALIDATION_ERROR", "fields": ["roomNumber"] }` |
| 409 | Room number already exists for this landlord's property | `{ "error": "ROOM_NUMBER_ALREADY_EXISTS" }` |

### rooms.API03 — GET /rooms/:id
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "roomId": "string", "roomNumber": "string", "size": "string", "company": "string", "owner": "string", "occupancy": "vacant" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Room not found, or not owned by caller | `{ "error": "ROOM_NOT_FOUND" }` |

### rooms.API04 — PUT /rooms/:id
**Request payload:**
```json
{ "roomNumber": "string", "size": "string", "company": "string", "owner": "string" }
```

**Success response (`200`):**
```json
{ "roomId": "string", "roomNumber": "string" }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 400 | Invalid/empty required field | `{ "error": "VALIDATION_ERROR" }` |
| 404 | Room not found, or not owned by caller | `{ "error": "ROOM_NOT_FOUND" }` |
| 409 | Room number collides with another room | `{ "error": "ROOM_NUMBER_ALREADY_EXISTS" }` |

### rooms.API05 — DELETE /rooms/:id
**Request payload:** _(none)_

**Success response (`200`):**
```json
{ "deleted": true }
```

**Exceptions:**

| Code | Condition | Response body |
| ---- | --------- | -------------- |
| 404 | Room not found, or not owned by caller | `{ "error": "ROOM_NOT_FOUND" }` |

## Acceptance Criteria

1. rooms.AC1 — Given a landlord caller, when `GET /rooms` is called, then only rooms owned by that landlord are returned, satisfying R23 scoping.
2. rooms.AC2 — Given no rooms exist for the caller, when `GET /rooms` is called, then an empty array is returned, not an error (R11).
3. rooms.AC3 — Given `?occupancy=occupied` or `?occupancy=vacant`, when `GET /rooms` is called, then only rooms matching that occupancy status are returned (R13).
4. rooms.AC4 — Given a unique room number, when `POST /rooms` is submitted, then a new room is created with default occupancy `vacant`, satisfying BRD-010.
5. rooms.AC5 — Given a room number that already exists for that landlord, when `POST /rooms` is submitted, then the request is rejected with `ROOM_NUMBER_ALREADY_EXISTS`, satisfying the "Room Number must be unique per property" validation rule.
6. rooms.AC6 — Given an existing room owned by the caller, when `PUT /rooms/:id` is submitted with valid fields, then the room is updated.
7. rooms.AC7 — Given a room not owned by the caller (or a `landlord` caller referencing another landlord's room), when `GET`/`PUT`/`DELETE /rooms/:id` is called, then the request is rejected with `404 ROOM_NOT_FOUND` (not `403`, to avoid confirming existence of another landlord's data).
8. rooms.AC8 — Given an existing room, when `DELETE /rooms/:id` is called, then only that specific room record is removed (R15) — no cascading deletion of unrelated rooms.
9. rooms.AC9 — Given a `super_admin` caller, when any `/rooms` endpoint is called, then it is not scoped to a single landlord — cross-landlord access is granted per the Role & Permission Matrix.

## Unit Test Cases (spec-derived)

| Test ID | Maps to AC | Scenario | Expected |
| ------- | ---------- | -------- | -------- |
| rooms.UT01 | AC1 | Landlord lists rooms | Only own rooms returned |
| rooms.UT02 | AC2 | Landlord with zero rooms lists rooms | 200, empty array |
| rooms.UT03 | AC3 | Filter by occupancy=occupied | Only occupied rooms returned |
| rooms.UT04 | AC4 | Create room with unique number | 201, occupancy=vacant |
| rooms.UT05 | AC5 | Create room with duplicate number | 409 ROOM_NUMBER_ALREADY_EXISTS |
| rooms.UT06 | AC6 | Update own room with valid data | 200, fields updated |
| rooms.UT07 | AC7 | Access another landlord's room by id | 404 ROOM_NOT_FOUND |
| rooms.UT08 | AC8 | Delete an existing room | 200 deleted=true; other rooms unaffected |
| rooms.UT09 | AC9 | Super-Admin lists rooms across landlords | Rooms from multiple landlords returned |

## Explicitly Out of Scope
- Linking a room to a Tenant/Agreement — the BRD's onboarding flow (BRD-006) links Tenant/Agreement to a Shop, not a Room; whether Rooms ever get tenants/agreements is unresolved (BRD Open Question #3: Shop vs. Room as the same underlying entity).
- Soft-delete / recoverability of deleted rooms — BRD Open Question #6, unresolved.

## Non-Functional Constraints (from constitution.md)
- Room Number must be unique per property (BRD §19 Validation Rules).
- Every request requires a valid session token (Security Posture, BRD §22).
- `landlordId` scoping derived from token, not request parameters (Security Posture, R23).
