# Pack 12 — Anti-Patterns Index

## When to load

- Something looks wrong but no scenario in ROUTER matches.
- Reviewing a diff and needing the "known ways this codebase breaks" list.
- Onboarding: skim once to know the failure modes.

## Scope / Out of scope

**Scope:** Symptom → root cause → fix index, compiled from the Forbidden /
Common Mistakes sections of every sub-repo rule. This pack is an **index**,
not a second rulebook — every entry links to its authoritative rule.

**Out of scope:** Everything else. For depth, always follow the linked rule.

## Web (financial-web)

| Symptom | Root cause | Fix / authority |
| ------- | ---------- | ---------------- |
| Import boundary violation in CI | Deep-import of private files, or `shared`→`core/api` / shell→domains | `pnpm deps:boundaries`; pack 02 barrel table |
| Rail icon visible, panel blank | Panel missing from `panelRegistry` | `financial-web/docs/rules/navigation-platform.md` |
| Menu never visible | Missing permission id / flag off / device mismatch | pack 04 common failures |
| CI permission mismatch | SSOT JSON edited, sync not run | `pnpm sync:navigation-permissions --check`; pack 04 |
| Deleted menu reappears after seed | `obsolete_node_ids` not updated in `app/db/seeders/mock.py` | pack 04 delete checklist |
| Raw i18n key shown in UI | Residual `fin_navigation_nodes` row (`kind: 'external'`) | pack 04; seed cleanup |
| `t` imported from `@/modules/i18n` inside `.vue` | Context model violation | pack 05 three-context table; `docs/rules/i18n.md` |
| Locale switch leaves stale WS items | Buffers not cleared / `idMatchesLocale` unused | pack 05 WS + locale alignment |
| Live UI polls REST with `setInterval` | Push feed implemented as polling | pack 05 refresh strategy; add WS channel instead |
| Second WebSocket client added casually | Isolation not actually required | pack 05 topology — add a channel, not a connection |
| Raw hex / px / ad-hoc breakpoint in scoped styles | Rule zero ignored | pack 09; `pnpm deps:style` |
| `var(--a, var(--b))` double fallback in component | Forbidden alias pattern | pack 09 layer model; `docs/rules/tokens.md` |
| Theme flashes light on first paint | JS fallback to light instead of stored/default dark | pack 09 runtime contract |
| Component sets `document.documentElement` / localStorage for theme | Bypassing `useTheme` / `useTrendColor` | pack 09 Forbidden |
| Config value hardcoded though an RCC key exists | L0 launch of an L1+ concern | pack 06; `financial/docs/rules/capability-activation.md` |
| Runtime-config endpoint called from a page | Must read via `getRuntimeConfig` with fallback | pack 06 non-negotiables |

## API (financial-api)

| Symptom | Root cause | Fix / authority |
| ------- | ---------- | ---------------- |
| Response `code: 1` on success | QuantDinger Human envelope leaked in | Red line #2; pack 03 envelope |
| Business logic in `router.py` / HTTP in `repository.py` | Layer model violated | pack 03 hard rules |
| Import alias (`from app.x import y as z`) | "No import aliases" rule broken | pack 03 |
| Parallel response model next to a domain schema | Normalizer invented a new model instead of mapping | pack 07 pipeline |
| Upstream fetched inside a router | Domain APIs must read PostgreSQL only | pack 07 boundary |
| Streaming job enqueued in arq | Process model violated (red line #7) | pack 07 three processes; `modules/worker.md` Forbidden |
| `realtime_hub` / `redis_bus` called from a crawl job | Push must go through `post_write.notify_*` after commit | pack 07; `modules/crawler/post-write.md` |
| Locale pollution in flash/calendar rows | `langId` trusted; content-encoded markers ignored | pack 07 bilingual; `uv run python -m app.crawler audit --heal` |
| JWT without `jti` | Local sessions unrevocable | pack 08 security layers; `modules/auth.md` Forbidden |
| Turnstile / rate limit bypassed on an auth endpoint | Security layers skipped | pack 08; `modules/auth.md` Forbidden |
| Audit written outside the business transaction | `audit_log()` not used with `db.flush()` semantics | red line #8; pack 08 audit discipline |
| Sensitive data in audit `before`/`after` | Masking bypassed (manual dict assembly) | pack 08; `modules/audit.md` |
| `admin_router.py` mounted under `/api/{domain}/admin/...` | Prefix convention broken (red line #19) | pack 08 RBAC |
| Admin metadata (`admin.*`, `navigation.*`) in anonymous snapshot | Key not `is_public=False` (red line #15) | pack 06; pack 03 |
| `print()` debug or bare `logging.getLogger(__name__)` | Logging discipline violated | pack 03 logging |
| Table/column missing `comment=` | ORM documentation rule skipped | pack 03 database rules |
| `Base.metadata.create_all()` on shared DB | Alembic bypass | pack 03 Forbidden |

## Cross-stack

| Symptom | Root cause | Fix / authority |
| ------- | ---------- | ---------------- |
| "Backend changed, frontend doesn't reflect it" | `pnpm api:sync` skipped / `core/api` module stale | pack 10 debugging sequence; pack 01 |
| Hand-edited `generated/*`, `schema.d.ts`, or topic-registry block | Generated-artifact ban ignored (red line #4) | pack 01 fan-out matrix |
| Stale `openapi.json` committed | Export not run after route change | pack 01 sequences; pack 11 |
| New domain missing from `CONTRACT.md` matrix | Matrix not maintained | pack 01 adding-a-domain |
| Only one side of a cross-stack change committed | Pair-update checklist skipped | pack 10 end-to-end checklist |
| camelCase/snake_case mixed across the wire | Field-naming convention broken | pack 10 envelope reminder |
| Secrets / `.env` in diff | Red line #1 violated | Remove immediately; pack 00 Do/Don't |

## Verify

None — this pack has no commands. Each row's authority column tells you which
rule or pack to load for the actual fix and its verification step.

## Related packs

- **00** — Workspace map (authority order)
- **11** — Verification (commands for every fix)
