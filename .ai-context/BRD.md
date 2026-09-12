# Business Requirements Document (BRD) Baseline

## Source
- **Document:** Rento BRD — Draft v1.0, 6 September 2026
- **Location:** `docs/Rento BRD.pdf`
- **Status:** Single authoritative source document (no competing versions found under `docs/`).
## Gate Approvals & History
- **2026-09-13 (Gate 0 BRD Review — BRD-Baseline-v1.1):** **APPROVED** by Supratim Jetty (`supratim.jetty@intglobal.com`). Feedback: *"Revised BRD baseline v1.1 and 7 backend business module boundaries (including Super-Admin RBAC amendment) formally approved at Gate 0."*. See dedicated review record: [GATE0-BRD-Baseline-v1.1-20260913-001915.md](file:///c:/Users/Supratim_Jetty/Desktop/office%20projects/AI_Projects/sdd-rento-ai/.ai-context/pr_reviews/GATE0-BRD-Baseline-v1.1-20260913-001915.md).
- **2026-09-13 (Gate 0 BRD Review — BRD-Baseline):** **REJECTED** by Supratim Jetty (`supratim.jetty@intglobal.com`). Feedback: *"Test the BRD properly and send back again"*. See dedicated review record: [GATE0-BRD-Baseline-20260913-000800.md](file:///c:/Users/Supratim_Jetty/Desktop/office%20projects/AI_Projects/sdd-rento-ai/.ai-context/pr_reviews/GATE0-BRD-Baseline-20260913-000800.md).
- **2026-09-13 (BRD Amendment — Role Management):** Added Super-Admin role and role-based access control at stakeholder request. This requirement is **not present in the source document** (`docs/Rento BRD.pdf`); it is a confirmed baseline amendment pending inclusion in a future client BRD revision. See [brd-change-log.md](brd-change-log.md) for the full delta and impact analysis. This change requires Gate 1 re-review before any dependent specs/architecture proceed.

## Objective
Rento lets a landlord who rents out commercial shops and residential rooms manage the full rental lifecycle from a single mobile app: onboarding a shop with its tenant and rental terms in one guided flow, tracking rooms/tenants/agreements, recording electricity-meter-based utility bills against a configurable rate, collecting rent and bill payments, formally closing out a shop when a tenancy ends, and monitoring the portfolio via a summary dashboard.

## Scope

### In Scope (of the Rento product; this repo builds the backend supporting it)
- Landlord-facing mobile app (Android + iOS) — built separately from this repository.
- Shop lifecycle: create, view, edit, close.
- Room lifecycle: create, view, edit.
- Tenant records linked to a shop/room.
- Rental agreement creation and history.
- Electric bill entry and a global service-rate configuration.
- Rent/bill payment capture and payment history.
- Portfolio dashboard with summary metrics.
- Account screens: login, sign-up, OTP verification, password recovery, profile, settings.
- **Role management (added 2026-09-13, see Gate Approvals & History):** Two fixed roles — **Landlord/Owner** (existing, unchanged: full access scoped to their own portfolio) and **Super-Admin** (new, platform-level: full CRUD access across every landlord's data plus landlord-account and role administration). Permissions are a fixed matrix per role — not user-configurable. See Actors & Role Management below.
- **This repository's scope:** the backend REST API, data persistence (PostgreSQL + Sequelize), and business logic serving all of the above.

### Out of Scope
- A tenant-facing login or app — tenants are records, not app users.
- Landlord-created staff/sub-accounts with custom or configurable permissions (e.g. a landlord inviting their own employees) — only the fixed Super-Admin and Landlord/Owner roles are in scope.
- Online payment gateway integration.
- Push, SMS, or email notification delivery.
- Multi-language / localization support.
- Web or desktop clients.
- Hosting infrastructure decisions beyond what's captured in `.ai-context/project_context.md` (Deployment Target: Unknown / TBD).

## Actors
| Role | Responsibilities | App Access |
|---|---|---|
| Super-Admin | Platform-level administrator. Creates and manages landlord accounts, assigns/revokes roles, and has full oversight (view + edit) of every landlord's shops, rooms, tenants, agreements, bills, and payments for support/administration purposes. | Full access to every module, across all landlord accounts. |
| Landlord / Property Owner | Adds and manages shops, rooms, tenants, agreements, billing, and payments; reviews the portfolio dashboard. | Full access to every module, scoped to their own account only. |
| Tenant | Occupies a shop or room under an agreement; contact and identity details are recorded against the shop/room they occupy. | None — no tenant-facing surface; data record only. |

### Role & Permission Matrix
Two fixed roles now exist — permissions are not user-configurable (Section 22, amended 2026-09-13):

| Module | Super-Admin | Landlord/Owner |
|---|---|---|
| Landlord Accounts | Create, view, edit, deactivate — any landlord account | View/edit own account only (via Profile) |
| Role Assignment | Assign/revoke roles for any account | None |
| Shops | View/Create/Edit/Close/Delete — all landlords | View/Create/Edit/Close/Delete — own only |
| Rooms | View/Create/Edit/Delete — all landlords | View/Create/Edit/Delete — own only |
| Tenants | View — all landlords | View; Create via shop onboarding — own only |
| Agreements | View — all landlords | View, Create — own only |
| Bills / Service Rate | View — all landlords | View, Create — own only |
| Payments | View — all landlords | View, Create — own only |
| Profile / Settings | Manage own Super-Admin profile | View, Edit own, Logout |

- Prior baseline ("a single Landlord role; no differentiated permission levels") is superseded by this amendment — see [Gate Approvals & History](#gate-approvals--history) and [brd-change-log.md](brd-change-log.md).
- Super-Admin's platform-wide visibility is **view/manage for oversight and account administration**; it does not change the day-to-day authoring flows, which remain landlord-scoped.

## Functional Requirements
Stable IDs below map 1:1 to the BRD's own Traceability Matrix (Section 33, `REQ01`–`REQ16`), preserved here as `BRD-001`–`BRD-016` for this project's SDD lifecycle. Do not renumber on future BRD revisions — see `.ai-context/brd-change-log.md`.

| ID | Requirement | Module | Data Entities |
|---|---|---|---|
| BRD-001 | Landlord authentication (login) | Authentication & Session | Landlord |
| BRD-002 | Landlord self-registration | Authentication & Session | Landlord |
| BRD-003 | Password recovery via OTP | Authentication & Session | Landlord |
| BRD-004 | Profile management | Account & Settings | Landlord |
| BRD-005 | Portfolio dashboard (summary metrics) | Dashboard | Shop, Payment, Agreement |
| BRD-006 | Shop creation wizard (Shop + Tenant + Agreement in one flow) | Shop Management | Shop, Tenant, Agreement |
| BRD-007 | Shop detail & sub-tabs (Information/Agreement/Payment/Settings) | Shop Management | Shop, Agreement, Payment |
| BRD-008 | Shop closure with settlement (deposit return, close date, remark, attachment) | Shop Management | Shop |
| BRD-009 | Shop search (by name or owner) | Shop Management | Shop |
| BRD-010 | Room management (CRUD) | Room Management | Room |
| BRD-011 | Tenant directory (list + detail) | Tenant Management | Tenant |
| BRD-012 | Agreement history & creation | Agreement Management | Agreement |
| BRD-013 | Electric bill calculation from meter readings | Billing Configuration | Electric Bill |
| BRD-014 | Global service-rate configuration (electric rate, late charge) | Billing Configuration | Service Rate Config |
| BRD-015 | Payment capture & receipt | Payments & History | Payment |
| BRD-016 | Payment history & combined activity log | Payments & History | Payment |
| BRD-017 | Role-based access control: Super-Admin and Landlord/Owner roles, landlord account administration | Access Control & Role Management | Landlord, Super-Admin |

*BRD-017 added 2026-09-13 — see Gate Approvals & History and [brd-change-log.md](brd-change-log.md). Not present in the original source document.*

## Backend API Contract (Section 27 — direct functional contract for this repository)
| Method & Path | Purpose |
|---|---|
| `POST /auth/login` | Authenticate, issue session token |
| `POST /auth/signup` | Register a landlord account |
| `POST /auth/otp/send` · `/verify` | Send and verify OTP |
| `POST /auth/password/reset` | Set a new password after OTP verification |
| `GET/PUT /profile` | Load and save the landlord profile |
| `GET /shops` · `POST /shops` | List and create shops |
| `GET/PUT /shops/:id` | Load and edit a shop |
| `POST /shops/:id/close` | Record a shop closure |
| `DELETE /shops/:id` | Delete a shop |
| `GET /rooms` · `POST /rooms` · `GET/PUT/DELETE /rooms/:id` | Room CRUD |
| `GET /tenants` · `GET /tenants/:id` | Tenant lookup (tenants are created via shop onboarding, not directly) |
| `GET /shops/:id/agreements` · `POST /agreements` | List and create agreements |
| `GET /shops/:id/bills` · `POST /bills/electric` | List and record bills |
| `GET/PUT /service-rate` | Global electric rate / late charge configuration |
| `POST /payments` · `GET /payments` | Record and list payments |
| `GET /dashboard/summary` | Metric values for the Home dashboard |
| `GET /admin/landlords` · `POST /admin/landlords` | Super-Admin: list and create landlord accounts |
| `GET/PUT /admin/landlords/:id` · `PUT /admin/landlords/:id/status` | Super-Admin: view/edit a landlord account, activate/deactivate |
| `PUT /admin/landlords/:id/role` | Super-Admin: assign/revoke a role |
| `GET /admin/shops` · `GET /admin/payments` · `GET /admin/dashboard/summary` | Super-Admin: cross-landlord read access for oversight (mirrors the landlord-scoped endpoints above, unscoped) |

## Business Data Model (Section 21)
- **Landlord** owns Shops and Rooms. Has a `role` attribute: `landlord` (default, own-scoped) or `super_admin` (platform-scoped, added 2026-09-13). Super-Admin accounts are still represented as Landlord records with the elevated role, not a separate top-level entity — unless architecture review determines a separate `Account`/`User` entity is warranted.
- **Shop**: email, shop type, description, medium, logo, name, rent/month, category. Has history of Agreements, is occupied by a Tenant, accrues Bills.
- **Room**: room number, size, company, owner, photo, attachment.
- **Agreement**: period, start date, end date, rent/month, pay period, security deposit, late charges, electrical flag, status, attachment.
- **Tenant**: name, father's name, address, email, phone, alternate phone, status, documents.
- **Electric Bill**: billing period, previous unit, current unit, rate, extra amount, amount, remark, attachment. Priced by Service Rate Config.
- **Payment**: bill type (Room Rent / Electric), amount, billing period, paid-on date, method (Cash/Online), remark, attachment. Settles a Bill.
- **Service Rate Config**: global Electric Rate and Late Rent Charge.

## Status & State Models (Section 20)
- **Shop lifecycle:** Active → Expired (agreement end date passes) → Active (renewed) ; Active/Expired → Closed (Close Shop confirmed) → deleted.
- **Payment/bill lifecycle:** Unpaid → Paid (Pay Now completed).
- **Room occupancy:** Occupied, Vacant.
- **Tenant status:** Active (occupying under a current agreement), Inactive.

## Non-Functional Requirements
| Area | Requirement |
|---|---|
| Responsive sizing | Consistent layout scaling across device sizes (client-side; informs API payload shaping, not backend NFR) |
| Session tokens | Stored in secure, encrypted on-device storage (client-side); backend issues and validates the token |
| Tenant PII | Names, addresses, phone numbers, and identity documents transmitted and stored **encrypted** |
| Payment data | Amounts and payment records protected **in transit and at rest** |
| API access | Every request beyond authentication requires a valid session token |
| [Open] | The BRD does not specify backend latency/throughput/availability targets, environment topology, or recovery objectives — these must be confirmed with the Technical Lead before being written into `.ai-context/constitution.md` as binding constraints. See Open Questions below. |

## Authentication & Authorization (Section 22)
| Concern | Requirement |
|---|---|
| Credential validation | Email and password checked against the registered account on login |
| Session issuance & storage | A session token is issued on successful login and stored securely on-device |
| Protected routes | Every screen/endpoint beyond Login/Sign Up/Forgot Password/OTP requires an active session |
| Token attachment | The session token is attached to every request made on the landlord's behalf |
| Session expiry | An expired or invalid session requires re-authentication |
| Logout | Clears the stored session and any cached account data |
| Roles | Two fixed roles: `landlord` (default, own-scoped) and `super_admin` (platform-scoped). Permissions are a fixed matrix per role, not user-configurable — see Actors & Role & Permission Matrix above. **Amended 2026-09-13** (prior baseline: a single "Landlord" role with no differentiated permission levels). |
| Authorization enforcement | Every protected endpoint checks the caller's role and, for `landlord`, scopes all reads/writes to that landlord's own records; `super_admin` is not scoped and can act across landlord accounts. |

This repository's confirmed strategy (`.ai-context/project_context.md`) is **JWT (access/refresh tokens)**, which satisfies the above.

## Business Rules
Numbered for traceability (Section 18), preserved verbatim from the BRD:

| ID | Rule |
|---|---|
| R01 | A modal dialog dims the background by 20% opacity by default. *(client-side)* |
| R02 | A dropdown selection is only committed when a non-empty value is chosen; clearing a dropdown does not commit a change. *(client-side)* |
| R03 | A primary action button defaults to the app's brand accent color unless a screen specifies otherwise. *(client-side)* |
| R04 | A button showing a loading state is disabled for the duration of that action, preventing duplicate submission. *(client-side)* |
| R05 | A date or time picker does not allow selecting a date before the current moment, unless the screen explicitly allows backdating. *(client-side, backend must enforce equivalent server-side)* |
| R06 | New Agreement and Add Electric Bill allow backdating up to 7 days, with no upper limit on future dates. |
| R07 | An agreement's start and end dates are selected independently, and the end date must fall after the start date. |
| R08 | Electric bill amount is calculated as `(Current Unit − Previous Unit) × Rate + Extra Amount`. |
| R09 | The current meter reading cannot be lower than the previous reading. |
| R10 | The configured Late Rent Charge is applied as a surcharge once rent becomes overdue. |
| R11 | Every list screen shows a "No Data Found" message when its underlying data is empty. *(client-side; backend returns empty arrays, not errors)* |
| R12 | Shop Search filters the shop list by name or owner as the search text is entered. |
| R13 | Rooms can be filtered by Occupied/Vacant, and Tenants by Active/Inactive. |
| R14 | Destructive actions — Logout, Close Shop, Delete Shop, Delete Room — require an explicit "Are you sure?" confirmation before proceeding. *(client-side confirmation; backend still performs the action on request)* |
| R15 | Deleting a shop or room removes that specific record and is scoped to the item selected. |
| R16 | Selecting a room, tenant, or shop from a list opens that record's detail or edit screen, pre-populated with its data. *(client-side)* |
| R17 | Login requires a registered email and matching password. |
| R18 | Logging out clears the stored session so the landlord must sign in again. |
| R19 | OTP verification checks the entered code against the one issued, and enforces a resend cooldown of approximately 75 seconds. |
| R20 | A new landlord can self-register for an account from the Sign Up screen. |
| R21 | The "Closed Shop Information" panel is shown on the Information tab only once a shop's status is Closed. *(client-side; backend must expose closure fields once status = Closed)* |
| R22 | Submitting a payment validates the amount, records the transaction, and carries its details through to the confirmation receipt. |
| R23 | *(Added 2026-09-13)* Every protected endpoint enforces role-based access: a `landlord` caller is restricted to records owned by their own account; a `super_admin` caller may access any landlord's records and administer landlord accounts/roles. |

## Validation Rules (Section 19)
| Field Category | Examples | Rule |
|---|---|---|
| Contact fields | Email, Phone, Alternate Phone | Format-validated; phone numbers are digit-only and length-bound |
| Monetary fields | Rent Amount, Security Deposit, Late Charges, Amount, Electric Rate | Positive numeric, currency-formatted |
| Meter readings | Previous Unit, Current Unit | Numeric; Current must be ≥ Previous |
| Date ranges | Start/End of Agreement, Billing Duration | End date must fall after start date; agreements for a shop must not overlap |
| Identifiers | Room Number | Must be unique per property |
| Required text | Name, Shop Type, Description, Remark | Must be non-empty before the form can be submitted |

## Error & Exception Handling (Section 29)
| Scenario | System Response |
|---|---|
| Invalid login credentials | Inline error message; landlord remains on Login |
| Empty list data | "No Data Found" message shown in place of the list |
| Network failure during a save/submit action | Error message with the option to retry |
| Invalid field input | Inline validation message next to the offending field |
| Session expiry mid-use | Redirect to Login with a re-authentication prompt |

## Assumptions
- Currency is Indian Rupees.
- Target platforms are Android and iOS only (mobile client — out of this repo's scope).
- The product is single-tenant-per-landlord-account.
- English is the only supported language.
- "Landlord" and "Super-Admin" (added 2026-09-13) are the only app-user personas; tenants are data subjects, not app users.

## Open Questions
1. What should the History screen show, distinct from Payment History?
2. What are the intended "medium" options in the Add Shop wizard's Step 1?
3. Are Shop and Room the same underlying rentable-unit entity, or genuinely separate types?
4. Should agreements support overlap checks — can two active agreements exist for one shop at once?
5. Is a tenant-facing app or portal planned for a future phase?
6. Should Delete Shop / Delete Room be recoverable (soft delete), and what happens to dependent agreements, bills, and payments?
7. **[Backend-specific, not in source BRD]** What are the backend's non-functional targets (p95 latency, availability, RPO/RTO)? Not specified in the BRD — must be confirmed before being codified in `.ai-context/constitution.md`.
8. **[Backend-specific, not in source BRD]** Deployment target is Unknown/TBD per `.ai-context/project_context.md` — confirm before first release.
9. **[Added 2026-09-13]** The source BRD's screen inventory has no Super-Admin console/UI. Is a Super-Admin-facing client in scope for a future phase, or is Super-Admin a backend/API-only role for now (e.g. operated via direct API calls or an internal tool)?
10. **[Added 2026-09-13]** Should Super-Admin actions (account creation, role changes, cross-landlord data access) be written to an audit trail? The BRD does not specify audit logging for admin actions.

## Acceptance Criteria
Acceptance Criteria are elaborated at the spec level (Gate 1) for each business module as it is developed — see `.ai-context/specs/<feature-slug>.spec.md`, one per module in the Traceability Matrix below. This BRD provides the requirement baseline; it does not itself carry AC-level detail beyond the Business/Validation Rules already listed above.

## Traceability Matrix
| Req. ID | Requirement | Module | Screen(s) (client, reference only) | Data Entity |
|---|---|---|---|---|
| BRD-001 | Landlord authentication | Authentication & Session | Login | Landlord |
| BRD-002 | Landlord self-registration | Authentication & Session | Sign Up | Landlord |
| BRD-003 | Password recovery via OTP | Authentication & Session | Forgot Password, OTP Verification | Landlord |
| BRD-004 | Profile management | Account & Settings | Edit Profile | Landlord |
| BRD-005 | Portfolio dashboard | Dashboard | Home | Shop, Payment, Agreement |
| BRD-006 | Shop creation wizard | Shop Management | Add Shop (3 steps) | Shop, Tenant, Agreement |
| BRD-007 | Shop detail & sub-tabs | Shop Management | Shop Details (4 tabs) | Shop, Agreement, Payment |
| BRD-008 | Shop closure with settlement | Shop Management | Close Shop, Settings tab | Shop |
| BRD-009 | Shop search | Shop Management | Shop Search | Shop |
| BRD-010 | Room management | Room Management | Rooms, Add Room, Edit Room | Room |
| BRD-011 | Tenant directory | Tenant Management | Tenants, Tenant Details | Tenant |
| BRD-012 | Agreement history & creation | Agreement Management | Agreements, New Agreement | Agreement |
| BRD-013 | Electric bill calculation | Billing Configuration | Add Electric Bill | Electric Bill |
| BRD-014 | Global rate configuration | Billing Configuration | Service Rate | Service Rate Config |
| BRD-015 | Payment capture & receipt | Payments & History | Pay Now, Confirmation | Payment |
| BRD-016 | Payment history | Payments & History | Payment History, History | Payment |
| BRD-017 | Role-based access control | Access Control & Role Management | *(no client screens — admin/back-office function; Super-Admin console is not in the source BRD's screen inventory)* | Landlord, Super-Admin |

## Glossary
| Term | Definition |
|---|---|
| Shop | A rentable commercial unit a landlord lets out, tracked with identity info, an occupying tenant, and rental terms. |
| Room | A separately-modeled rentable unit for residential letting. |
| Agreement | A lease/rental contract record for a shop — period, dates, rent, deposit, late-charge terms, and status. |
| Tenant | The occupant of a shop or room; a data record, not an app user. |
| Electric Bill | A billing-period record of electricity consumption and its calculated cost. |
| Service Rate | Landlord-configured defaults: per-unit electric rate and late-rent-charge amount. |
| OTP | One-time password used for identity/password-recovery verification. |
| KPI | Key Performance Indicator — the summary metrics on the Home dashboard. |
| Super-Admin | *(Added 2026-09-13)* Platform-level role with full CRUD access across every landlord's data plus landlord-account and role administration. |
| RBAC | Role-Based Access Control — the fixed Super-Admin / Landlord permission model enforced on every protected endpoint. |
