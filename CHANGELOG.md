# Changelog

All notable changes to this project will be documented here.  
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

---

## [1.2.0] — 2026-07-06

### Added
- **Data Profiling: Schema, Shape & Volume** (`16_data-profiling.md`) — 3-layer model:
  - Schema (نظام البيانات): drift detection via versioned schema snapshots, 4-source contract verification (DB = ORM = API = client types), enum closure, orphaned-reference checks for soft-FK systems (Convex)
  - Shape (شكل البيانات): per-table statistical profiles (null %, cardinality, distributions), auto-finding red-flag table (seed leaks, overflow boundaries, backfill clusters, Arabic digit/normalization issues), versioned profile diffing per release
  - Volume (حجم البيانات): growth-rate audit, EXPLAIN ANALYZE on hot queries, 10x scale testing with profile-matched synthetic seeding, OFFSET-death pagination test, realtime fan-out and cron-window checks at scale
- New routing mode: `data-profiling` (Arabic + English triggers)
- Sequencing rule: 16 (structure) → 15 (values) → 09 (performance) → 06 (gate); wired into Super Power dependency rules

---

## [1.1.0] — 2026-07-06

### Added
- **Super Power orchestrator mode** (`13_super-power.md`) — mission board, phase sequencing with dependency rules, checkpoints, halt/escalation conditions, consolidated Master Report with single GO/NO-GO
- **Smart Planning & Traceability** (`14_smart-planning.md`) — risk-based planning (impact × likelihood, 60/30/10 effort rule), Requirements Traceability Matrix (RTM) with forward/backward trace rules, coverage metric, drift event log, path-integrity checkpoints (ON-PATH / DRIFTED-CORRECTED / OFF-PATH)
- **Data Quality & Accuracy** (`15_data-quality.md`) — 6-dimension model with weighted DQ Score, SQL/Supabase/Convex check library, RLS cross-tenant leak verification, Arabic/RTL data validity checks, migration/pipeline checklist
- 3 new routing modes: `super-power`, `smart-plan`, `data-quality` (Arabic + English triggers)
- 3 new universal principles: plan by risk, nothing untraced ships, correct data beats working UI

### Changed
- Skill description expanded to trigger on planning, traceability, and data-quality requests
- `data-quality` sequenced before `security-scan` in orchestrated missions

---

## [1.0.0] — 2026-07-05

### Added
- 12 testing modes with auto-routing (Arabic + English triggers)
- Stack detection for Next.js, Flutter, Supabase, Convex, Prisma, Drizzle, tRPC, Clerk, Bun
- OWASP Top 10 security checklist (all 10 — A01–A10)
- E2E testing with Playwright + Cypress (Page Object Model, GitHub Actions CI)
- API contract testing with Postman/Newman + OpenAPI/Spectral validation
- Channel integration: Discord webhooks, Jira REST API v3, WhatsApp formatted templates
- Emergency push for S1/Security Critical findings (`@here` + Jira Highest)
- RTL/Arabic as first-class testing concern across all reference files
- 5-gate release gate with GO/NO-GO verdict
- Configurable `OWNER_NAME` — works for any individual or team
- MIT License
