# Architecture

## Overview
Rento is a Backend Only project using a **Monolithic** architecture: a single deployable Node.js + Express service backed by PostgreSQL via Sequelize.

## Execution Layer Structure

```text
src/
└── backend/
    ├── app/
    │   ├── config/
    │   ├── middleware/
    │   ├── routes/
    │   └── server.js
    ├── modules/
    └── shared/
        ├── database/
        ├── logger/
        ├── errors/
        └── utils/
tests/
└── backend/
    ├── config/
    ├── modules/
    └── shared/
docs/
```

- `app/` — application bootstrap: server entrypoint, route mounting, middleware, and configuration loading.
- `modules/` — business feature modules. Each spec-derived feature lives here as it's implemented.
- `shared/` — cross-cutting infrastructure: database connection/models, logging, error types, and utilities.
- `tests/backend/` mirrors the `config/`, `modules/`, and `shared/` hierarchy of `src/backend/`.

## Data Access
- Database: PostgreSQL.
- ORM: Sequelize.
- Migrations and models live under `src/backend/shared/database/`.

## Proposed Business Module Architecture (Pending Gate 1 Approval)

Derived from BRD analysis (`.ai-context/BRD.md`, Section 06 Module Overview). Per the `int-brd-ingestion` workflow, these are **proposed** module boundaries only — `src/backend/modules/<name>/` folders are NOT generated until this proposal receives Gate 1 (Spec Peer Review / Architecture Review) approval.

The BRD's 9 client-facing modules are grouped into fewer backend-owned modules where the underlying data and lifecycle are tightly coupled (notably: Tenant is created only via Shop onboarding, and Agreement is a sub-resource of Shop), while modules with independent lifecycles remain separate:

| Proposed Backend Module | Covers BRD Modules | Rationale | Primary Entities |
|---|---|---|---|
| `auth` | Authentication & Session | Independent lifecycle (session/credentials), no shared entities with other modules | Landlord (auth fields) |
| `account` | Account & Settings | Profile CRUD is independent of shop/room domain logic | Landlord (profile fields) |
| `dashboard` | Dashboard | Read-only aggregation across Shop/Payment/Agreement — kept separate so it stays a thin reporting layer, not a place business rules leak into | (aggregates only, no owned entity) |
| `shops` | Shop Management, Tenant Management, Agreement Management | Tenant is only ever created through shop onboarding (BRD-006); Agreement is always scoped to a shop (`GET /shops/:id/agreements`). Splitting these into separate modules would force circular dependencies for a single onboarding transaction (BRD rule: shop + tenant + initial agreement created together). Tenant/Agreement sub-resources may still get their own `services/`/`repositories/` files within this module for internal separation. | Shop, Tenant, Agreement |
| `rooms` | Room Management | Independent CRUD lifecycle, no dependency on Shop | Room |
| `billing` | Billing Configuration | Global Service Rate config + Electric Bill calculation form one cohesive billing concern | Electric Bill, Service Rate Config |
| `payments` | Payments & History | Payment capture/history is a distinct concern from bill calculation, settles bills from either `shops` (room rent) or `billing` (electric) | Payment |

Cross-module rule (Microservice Readiness, per `int-brd-ingestion` guidance): `payments` and `billing` reference `shops`/`rooms` by ID only, never via direct DB joins across module boundaries, so each could later be extracted independently.

**Open architectural question carried from the BRD (Section 35, Q3):** the BRD leaves it unclear whether Shop and Room are the same underlying rentable-unit entity or genuinely separate types. This proposal treats them as separate modules/entities (matching the BRD's separate CRUD screens and separate API paths `/shops` vs `/rooms`); reversing this would be a breaking architectural change requiring an ADR.

## Authentication & Security
- JWT-based authentication (access/refresh tokens).
- Secrets and credentials are read from environment variables only; never committed to source control.

## Deployment
- Target: Unknown / TBD. Update this section once a deployment target (Docker, AWS, etc.) is confirmed.

## Change Log
Architectural decisions that alter this baseline must be recorded as an ADR under `.ai-context/decisions/ADR-NNN.md` and reflected here.
