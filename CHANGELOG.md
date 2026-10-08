# Expert Pack Changelog

> Tracks pack-level changes. Follows Keep-a-Changelog format.

## [0.7.0] — 2026-10-08

### Added — P1 packs delivered (04–12, previously repo-fallback only)

Distilled from the sub-repo rules (re-verified against the 2026-10-08
rules/skills overhaul), following the P0 pack structure:

- **04 Navigation + Manifest** — ownership layers, two sync tracks (Manifest
  codegen + permissions parity), add/modify/delete checklists, failure table.
- **05 Realtime + i18n + Data Sources** — REST vs WS strategy, single-client
  multiplexed topology, i18n three-context model, copy ownership, locale
  propagation (`Accept-Language` + `?lang=`), WS lang alignment.
- **06 RCC + Capability** — snapshot pipeline + read priority, Feature-L vs
  Platform-L, maturity L0–L4, new-key 5-step checklist.
- **07 Crawler + Scripts + Worker** — boundary, pipeline, L1–L4 scheduler,
  enqueue-only three-process model, post-write broadcast, bilingual ingestion,
  dynamic-scripts platform, arq/streaming Forbidden.
- **08 Auth + Audit** — dual-backend auth, security layers, RBAC capabilities,
  `audit_log()` same-transaction discipline, endpoint inventory.
- **09 Styles + Tokens + Theme** — rule zero + token selection order, layer
  model, forbidden aliases, theme/trend runtime contract, component template.
- **12 Anti-Patterns** — symptom → cause → authority index (web / api /
  cross-stack), compiled from every rule's Forbidden sections.

### Changed

- `ROUTER.md`: P1 table marked delivered ("Authoritative rule" column);
  removed "not yet delivered" notices.
- `README.md`: structure tree and delivery status updated; delivery date
  bumped to 2026-10-08.
- Packs 00/02/03/10: "(0x when available)" placeholders replaced with direct
  pack references.

## [0.6.0] — 2026-10-08

### Fixed — dead links after upstream rules/skills overhaul

Sub-repos `financial-web` and `financial-api` underwent a full rules/skills
restructuring (2026-10-08: unified skill file structure standard, doc gate
upgrades, AGENTS.md single-editor entry). Pack references re-verified against
the repos; stale entries corrected:

- **pack 00 / pack 10:** removed SSOT links to
  `financial/docs/superpowers/2026-08-11-expert-scheme/` — the expert-scheme
  directory was deleted upstream; the chronicle record lives in
  `financial/docs/reviews/README.md` (cycles 5 & 7).
- **ROUTER P1 fallback (08 Auth + Audit):** `docs/rules/modules/auth/` →
  `docs/rules/modules/auth.md` (file, not directory).
- **pack 02:** `src/shared/ui` sublayers `common / business / widgets` →
  `common / messaging / widgets` (`business/` was removed upstream; its
  components moved to `common/widgets/messaging` or `domains/*/components`).
  New-shared-component sequence updated accordingly.

### Corrected

- Red line count: IDENTITY.md carries **20** red lines; README/SKILL.md said
  18. Numbering aligned everywhere.

### Verified still-valid (no change needed)

- All web top-level rules paths (`core.md`, `architecture.md`, `i18n.md`,
  `navigation-platform.md`, etc.), api rules paths (`core/*`,
  `modules/database/*`, `modules/auth.md`, `modules/crawler/`), admin rules.
- `docs/skills/project/` inventories: web 26 packs, api 18 packs — ROUTER
  fallback targets all exist.
- pnpm commands in pack 01/02/11 (`check`, `deps:*`, `api:sync:check`,
  `codegen:*:check`, `sync:*`) all present in `financial-web/package.json`.
- Web doc gate `pnpm deps:rules-docs` (= `node scripts/check-rules-docs.mjs`)
  unchanged; api repo (Python) has no package gate — unchanged.

## [0.5.0] — 2026-08-18

### Corrected — Red lines #16–18 (post ruleskills audit)

After applying ruleskills-kit "Project First" methodology (reading project's
own `docs/rules/**` as SSOT before applying generic guidance), the previous
v0.4.0 wording for red lines #16-18 was **overstated**:

- **#16 (was: "four tables MUST exist"):** Corrected. The tables are designed
  in the control-plane doc (Stage B, no hard deadline). The real rule is the
  **admin-capabilities sync mechanism** (`pnpm sync:admin-capabilities`),
  which IS mandatory per `modules/admin-capabilities.md`.
- **#17 (was: "table names must be plural"):** Corrected. Plural naming is a
  convention/example in `design-principles.md`, not a hard rule for existing
  tables. The hard rule is `TimestampMixin` inheritance + correct seed
  helper usage per `modules/database/seed.md`.
- **#18 (was: "config tables use upsert, content tables use insert-only"):**
  Corrected. `modules/database/seed.md` explicitly defines **"Insert + Purge
  Obsolete"** as a valid strategy for pure fixture-driven config tables.
  The real rule: seed must be idempotent + purge-safe, use the documented
  helpers, and never use Wipe-and-Reinsert.

### Added — SSOT methodology note + execution plan

- `financial/docs/reviews/2026-08-18-supplementary-refactoring-plan.md` now
  carries a "Methodology" header citing ruleskills-kit Project First.
- `financial/docs/reviews/2026-08-18-unified-execution-plan.md` — canonical
  Phase 0-6 execution sequence (60 person-days, 11 weeks), the authoritative
  sequencing input for Leader approval.

### Changed

- IDENTITY.md red lines #16-18 rewritten for accuracy against project rules.

## [0.4.0] — 2026-08-18

### Added — Red lines #16–18

- **#16 (Admin control-plane tables):** Four tables must exist before Stage B:
  `fin_admin_role_capabilities`, `fin_config_revisions`,
  `fin_role_permission_bindings`, `fin_admin_sessions`. RCC JSON is a shim.
- **#17 (TimestampMixin + plural table names):** All CMS tables inherit
  `TimestampMixin`; no hand-written timestamps; table names pluralized
  (`fin_headlines`).
- **#18 (Seed safety):** Seed must be idempotent + purge-safe;
  `SKIP_SEED_PURGE` default-on in prod; no hardcoded weak passwords;
  config tables upsert, content tables insert-only.

### Added — ROUTER.md scenario rows

- `Admin control-plane tables / config revisions / admin sessions` → packs 03, 11
- `Seed data safety / purge / weak-passwords / strategy consistency` → packs 03, 11
- `Database table design (TimestampMixin, plural names, JSONB cleanup)` → packs 03, 11
- `Admin Dashboard product-health / debug-push / crawler write ops` → packs 00, 11

### Added — SSOT references

- `financial/docs/reviews/2026-08-18-admin-database-seed-supplement.md`
  (Admin control-plane + DB tables + Seed governance supplement)

### Changed

- IDENTITY.md red lines expanded from 15 to 18.
- SKILL.md now references 18 red lines.

## [0.3.0] — 2026-08-18

### Added — Red lines #12–15

- **#12 (shell→domains ban):** Shell must never `import @/domains/**`.
  All auth/session/home access via bootstrap providers only. Drives from
  code-audit finding: 6 direct shell→domains imports found, breaking the
  "bootstrap is the only cross-layer bridge" rule.
- **#13 (shared God-object ban):** Shared must never `import @/core/api`
  or `import @/domains/**`. 35+ violations found. Fetachable UI stays in
  shared; data-fetching UI moves back to domains.
- **#14 (startup migrate/seed ban):** API must not auto-run `alembic upgrade`
  or full seed in production startup. `MIGRATE_ON_STARTUP`/`SEED_ON_STARTUP`
  must default to off in prod. Traces to architecture review A-P0-04 and
  QA P1-11.
- **#15 (admin metadata is_public=False):** Admin/policy metadata must be
  `is_public=False`. Traces to architecture review A-P0-01, QA findings,
  and the supplementary refactoring plan.

### Added — ROUTER.md scenario rows

- `Shell→domains coupling / shared God-object cleanup` → packs 02, 11
- `Seed data / JSONB array → row-level table migration` → packs 03, 11
- `Startup safety (migrate/seed-on-start, scheduler lease)` → packs 03, 11
- `Admin metadata isolation (is_public=False)` → packs 03, 11
- `Control-plane audit coverage gap` → packs 03, 11
- `Supplementary refactoring plan (all non-compliant designs)` → packs 02, 03, 11

### Added — SSOT references

- `financial/docs/reviews/2026-08-18-supplementary-refactoring-plan.md`
  (consolidated non-compliant design inventory)
- `financial/docs/reviews/2026-08-18-seed-data-governance-redesign.md`
  (JSONB → row-level migration)

### Changed

- SKILL.md now references 15 red lines (was 11).
- IDENTITY.md red-line table expanded from 11 to 15 entries.

## [0.2.0] — 2026-08-11

### Changed

- Removed red line #12 (dual-source checklist ban) — was too restrictive;
  project `docs/skills/project` are now complementary, not duplicate.
- Renumbered red lines from 12 to 11.
- Replaced all absolute IDE-specific paths (`D:\Workspace\.cursor\rules\`,
  `D:\Workspace\.catpaw\rules\`) with generic "workspace rules" references.
- Updated authority order: `docs/skills/project` repositioned as
  "complementary, not duplicate" instead of "only when pack does not cover".
- Replaced all `[Planned]` / `when available` markers in ROUTER.md with
  concrete repo fallback links for each P1 pack.
- Updated SKILL.md to reference 11 red lines (was 12).
- Updated README.md to list P1 packs with ⏳ status and repo fallback mapping.

### Added — P2 (tool adapters)

- `adapters/claude-code.md` — CLAUDE.md pointer template
- `adapters/codex.md` — AGENTS.md pointer template
- `adapters/chatgpt.md` — ChatGPT Projects upload guide

### Fixed

- All conflict priority blocks now use generic "workspace rules" instead of
  hardcoded IDE-specific paths.
- `adapters/cursor.md` no longer hardcodes `.cursor/rules/` as the only
  workspace rules directory.

## [0.1.0] — 2026-08-11

### Added — P0 (core orientation)

- `IDENTITY.md` — always-loaded identity, 12 red lines, `code: 0` mnemonic,
  three-process model, conflict priority
- `ROUTER.md` — scenario → pack routing table (≥25 P0-relevant rows),
  business pack budget ≤4, P1 packs marked Planned
- `packs/00-workspace-map.md` — three-repo structure, path anchors, three
  processes, authority order, deepquant boundary
- `packs/01-contracts-ssot.md` — contract fan-out matrix, domain matrix,
  sync/check commands, generated artifact ban
- `packs/02-web-architecture.md` — web layering, `@/core/api` boundary, 22
  modules table, barrel/i18n summary
- `packs/03-api-architecture.md` — Bounded Context table, layer model, pkgutil
  discovery, seed/fixtures touchpoints, no import alias
- `packs/10-fullstack-playbook.md` — end-to-end pair-update checklist,
  domain classification, verification matrix
- `packs/11-verification.md` — web `pnpm check` / `deps:*` / `codegen:*:check`,
  api `ruff` + `pytest`, admin three commands, WSL vs PowerShell
- `adapters/generic-system-prompt.md` — universal system prompt (identity +
  red lines summary + key paths)
- `adapters/cursor.md` — Cursor rules / skill forwarding (≤80 lines)

### Notes

- Old `docs/experts/` (plural) is **DELETED** — never revive.
- Scheme and acceptance docs (formerly in `docs/superpowers/2026-08-11-expert-scheme/`)
  are **DELETED** — the delivered packs are the SSOT now.
- P1 packs (04–09, 12) are planned but not yet delivered. Fall back to sub-repo
  `docs/rules/**` and `docs/skills/project/**` for those scenarios.
