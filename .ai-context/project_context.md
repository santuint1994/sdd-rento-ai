# Project Context

## Project Name
Rento

## Project Type
Backend Only

## Architecture Style
Monolithic

## Backend Technology & Framework
Node.js + Express

## Database & Data Access / ORM Layer
PostgreSQL + Sequelize

## Authentication & Security Strategy
JWT (access/refresh tokens)

## Deployment Target
Unknown / TBD

## Gate 1 Reviewer(s)
Supratim Jetty (supratim.jetty@intglobal.com)

## Gate 2 Reviewer(s)
Supratim Jetty (supratim.jetty@intglobal.com)

## BRD Status
Ingested. Source: `docs/Rento BRD.pdf` (Rento BRD — Draft v1.0, 6 September 2026). Authoritative baseline: `.ai-context/BRD.md`. See `.ai-context/brd-change-log.md` for ingestion history.

Proposed backend business module boundaries (`auth`, `account`, `dashboard`, `shops`, `rooms`, `billing`, `payments`) are documented in `.ai-context/architecture.md` and await Gate 1 architecture approval before `src/backend/modules/<name>/` folders are generated.

## Notes
- No frontend is part of this repository's scope.
- Deployment target is undecided; revisit before the first release and update `.ai-context/constitution.md` and `.ai-context/releases/` accordingly.
