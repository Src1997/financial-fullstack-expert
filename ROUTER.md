# Scenario Router

> Look up your scenario, load the listed packs, then code.
> Always-loaded: `IDENTITY.md` + this file.
> Business pack budget: **≤ 4** per session (excluding always-loaded).

## Available packs

### P0 (core orientation)

| Pack | Title |
|------|-------|
| 00 | Workspace Map |
| 01 | Contracts SSOT |
| 02 | Web Architecture |
| 03 | API Architecture |
| 10 | Fullstack Playbook |
| 11 | Verification |

### P1 (domain deep dive)

| Pack | Title | Authoritative rule |
|------|-------|--------------------|
| 04 | Navigation + Manifest | `docs/rules/navigation-platform.md` |
| 05 | Realtime + i18n | `docs/rules/data-sources.md` + `docs/rules/i18n.md` |
| 06 | RCC + Capability | `docs/rules/runtime-config.md` + `docs/rules/capability-activation.md` |
| 07 | Crawler + Scripts + Worker | `docs/rules/modules/crawler/` + `docs/rules/modules/worker.md` |
| 08 | Auth + Audit | `docs/rules/modules/auth.md` + `docs/rules/modules/audit.md` |
| 09 | Styles + Tokens + Theme | `docs/rules/styles.md` + `docs/rules/tokens.md` + `docs/rules/theme.md` |
| 12 | Anti-Patterns (index) | every entry links its authoritative rule |

> P1 packs are distilled summaries. For depth, follow the pack's SSOT links
> to the repo docs listed above.

## Adapters (P2)

| Adapter | Tool |
|---------|------|
| `adapters/generic-system-prompt.md` | Any AI tool with system prompt |
| `adapters/cursor.md` | Cursor (rules / skill forwarding) |
| `adapters/claude-code.md` | Claude Code (`CLAUDE.md`) |
| `adapters/codex.md` | Codex / AGENTS.md-aware tools |
| `adapters/chatgpt.md` | ChatGPT Projects / Custom GPTs |

## Scenario routing table

| Scenario | Must load | Strongly related |
|----------|-----------|------------------|
| Cold start (any task) | IDENTITY, ROUTER, 00 | — |
| End-to-end new feature | 01, 02, 03, 10 | 11 |
| OpenAPI / type sync | 01, 11 | 03 |
| Realtime topics change | 01, 11 | 02 |
| Admin capabilities change | 01, 11 | — |
| Add/remove C-end menu | 01, 11, 02 | 04 |
| Shell / Chrome behavior | 02 | 04, 09 |
| Home widget / page_layout | 02, 01 | 04, 06 |
| RCC new key | 11, 01 | 06 |
| Feature vs Platform capability | 01 | 06, 08 |
| WS push / subscribe | 02, 01 | 05 |
| WS push wrong language | 01, 02 | 05 |
| Add new locale | 02 | `i18n.md` + api `i18n-adding-locales.md` |
| New FastAPI domain | 03, 01, 10, 11 | — |
| New table / migration / seed | 03, 11 | — |
| Crawler source / FastBull | 11 | 07 |
| Dynamic script | 11 | 07, 08 |
| arq / streaming worker | 11 | 07 |
| MCP / external market source | 01 | 07 |
| Login / guard / RBAC | 02 | 08, 04 |
| Control-plane write + audit | 01 | 08, 06 |
| SCSS / tokens | 02 | 09 |
| Theme / trend colors | 02 | 09 |
| Responsive three-track | 02 | 04, 09 |
| Commenting / logging conventions | 02 or 03 + 11 | `commenting` / `logging` rules |
| Admin new page (placeholder) | 00 + admin rules | — |
| "Why only one side changed" | 10, 01 | — |
| Self-check before blind change | 11 | ROUTER |
| Pre-PR / release gate | 11 | — |
| Shell→domains coupling / shared God-object cleanup | 02, 11 | `financial-web/docs/rules/core.md` + `architecture.md` |
| Seed data / JSONB array → row-level table migration | 03, 11 | `financial/docs/reviews/2026-08-18-seed-data-governance-redesign.md` |
| Startup safety (migrate/seed-on-start, scheduler lease) | 03, 11 | `financial-api/docs/rules/modules/database/seed.md` + `modules/worker.md` |
| Admin metadata isolation (is_public=False) | 03, 11 | `financial-api/docs/rules/modules/app_config.md` |
| Control-plane audit coverage gap | 03, 11 | `financial-api/docs/rules/modules/audit.md` |
| Supplementary refactoring plan (all non-compliant designs) | 02, 03, 11 | `financial/docs/reviews/2026-08-18-supplementary-refactoring-plan.md` |
| Admin control-plane tables / config revisions / admin sessions | 03, 11 | `financial/docs/reviews/2026-08-18-admin-database-seed-supplement.md` + `docs/rules/modules/database/` |
| Seed data safety / purge / weak-passwords / strategy consistency | 03, 11 | `financial/docs/reviews/2026-08-18-admin-database-seed-supplement.md` + `docs/rules/modules/database/seed.md` |
| Database table design (TimestampMixin, plural names, JSONB cleanup) | 03, 11 | `financial/docs/reviews/2026-08-18-admin-database-seed-supplement.md` + `docs/rules/modules/database/table-design.md` |
| Admin Dashboard product-health / debug-push / crawler write ops | 00, 11 | `financial/docs/reviews/2026-08-18-admin-database-seed-supplement.md` |

## Loading tips

- If the scenario does not match any row, start with **00 + 01 + 11** and Read
  the relevant sub-repo `docs/rules/README.md`.
- For cross-stack changes, always load **pack 10** (Fullstack Playbook) as a
  pair-update checklist companion.
- When debugging unknown issues, check the sub-repo `docs/rules/**` and
  `docs/skills/project/**` listed in the SSOT links of each pack.
- P1 packs (04–12) are distilled summaries of the authoritative rules —
  follow their SSOT links when the summary is not enough.
