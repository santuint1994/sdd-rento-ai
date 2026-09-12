# BRD Change Log

## 2026-09-12 — Initial BRD Ingestion
- **Source Document:** `docs/Rento BRD.pdf` (Rento BRD — Draft v1.0, 6 September 2026)
- **Change Type:** Initial ingestion (no prior BRD existed).
- **Summary:** Ingested the full Rento product BRD covering 9 functional modules (Authentication & Session, Account & Settings, Dashboard, Shop Management, Room Management, Tenant Management, Agreement Management, Billing Configuration, Payments & History), 16 functional requirements (BRD-001–BRD-016), 22 business rules (R01–R22), the business data model, and the backend API contract (Section 27).
- **Scope Note:** The BRD describes the full Rento mobile app product; this repository implements only the backend (per `.ai-context/project_context.md`: Backend Only, Monolithic, Node.js + Express, PostgreSQL + Sequelize, JWT). The BRD's Section 27 (API & Integration Requirements) is treated as this repository's direct functional contract.
- **No Constitution section found in the BRD** — `.ai-context/constitution.md` retains the INT baseline structure, enriched with the BRD's Security Requirements (Section 31) and Non-Functional Requirements (Section 30) where applicable. Backend-specific NFRs (latency, availability, RPO/RTO) are not specified in the BRD and are logged as Open Questions pending Technical Lead confirmation.
- **Impact:** No existing specs/architecture to re-review (first ingestion). Proposed backend business module boundaries were added to `.ai-context/architecture.md` pending Gate 1 architecture approval before any `src/backend/modules/<name>/` folders are generated.

## 2026-09-13 — Amendment: Role Management (Super-Admin + Landlord/Owner)
- **Source:** Direct stakeholder request in-session, **not from `docs/Rento BRD.pdf`**. The source document explicitly scoped this out (Out of Scope: "Multi-landlord or staff accounts with role-based permissions"; Section 22: "A single Landlord role; no differentiated permission levels"). This is a confirmed baseline amendment pending reconciliation with a future client BRD revision.
- **Change Type:** Requirement addition (new actor, new functional requirement, scope change).
- **Decisions confirmed with stakeholder (2026-09-13):**
  - Two fixed roles only: `landlord` (existing, own-scoped, unchanged permissions) and `super_admin` (new, platform-scoped).
  - Super-Admin manages landlord account creation/status and role assignment.
  - Super-Admin has **full platform access** — full CRUD across every landlord's shops, rooms, tenants, agreements, bills, and payments — not just read-only oversight.
  - Permission model is a **fixed matrix per role**, not user-configurable custom RBAC.
  - Landlord-created staff/sub-accounts (a third role tier) were explicitly declined and remain Out of Scope.
- **Delta Summary:**
  - Added: BRD-017 (Role-based access control), Actor "Super-Admin", `role` attribute on Landlord entity, R23 (RBAC enforcement rule), 5 admin API endpoints, 2 Open Questions (Super-Admin client surface, admin audit logging).
  - Modified: Scope (In Scope / Out of Scope), Actors & Role & Permission Matrix (was single-role, now two-role matrix), Authentication & Authorization Roles row, Assumptions (persona list), Traceability Matrix.
  - Removed: None — no existing requirement IDs were renumbered or deleted; the prior single-role statements are marked superseded, not deleted, for auditability.
- **Impact:**
  - **Gate 1 re-review required** before any dependent specs or architecture proceed, per BRD Change Management Process — this changes Actors, Scope, and adds a new functional requirement.
  - `.ai-context/architecture.md` should be checked for a new "Access Control & Role Management" module boundary and for whether existing module boundaries need a role-scoping/authorization layer added.
  - `.ai-context/constitution.md` Security Posture / Architectural Constraints should be reviewed for RBAC enforcement guidance (e.g. middleware pattern for `landlord`-scoping vs `super_admin`-unscoped access).
  - No existing specs exist yet under `.ai-context/specs/` for this project, so no spec-level rework is required at this time.
  - **Open risk:** the source client BRD document has not been updated to reflect this change — recommend requesting a revised BRD from the client to keep `docs/` and `.ai-context/BRD.md` reconciled, per the BRD Ingestion process.
