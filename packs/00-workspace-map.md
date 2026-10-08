# Pack 00 — Workspace Map

## When to load

- **Always** during cold start — this is the foundational orientation pack.
- Load alongside `IDENTITY.md` + `ROUTER.md` for any task.

## Scope / Out of scope

**Scope:** Three-repo layout, technology stack, path anchors, backend
three-process model, authority order, deepquant boundary.

**Out of scope:** Contract details (→ pack 01), web layering deep dive (→ pack
02), API domain design (→ pack 03), verification commands (→ pack 11).

## Must know

### Three repositories (each independent Git)

| Repo | Role | Path |
|------|------|------|
| `financial-web` | Vue 3.5 C-端 frontend; **contracts SSOT host** | `financial/financial-web/` |
| `financial-api` | FastAPI backend; crawler / realtime / worker / DB | `financial/financial-api/` |
| `financial-admin` | Admin SPA (Vue 3); capability codegen consumer | `financial/financial-admin/` |

> Root `financial/` is a workspace container, not a monorepo. Commit inside
> each repo independently.

### Technology stack

| Layer | Technology | Path anchor |
|-------|-----------|-------------|
| C-端 frontend | Vue 3.5, TS, Vite, Pinia, Vue Router, Element Plus, SCSS, pnpm 11.5.1 | `financial-web/` |
| Contracts | JSON/YAML SSOT + codegen/sync | `financial-web/packages/contracts/` |
| Backend | Python 3.11+, FastAPI, Pydantic v2, SQLAlchemy 2.x, uv | `financial-api/` |
| Database | PostgreSQL (`quant_zc`); Redis; Alembic migrations | `financial-api/app/db/` |
| Realtime | WebSocket hub, Redis bus, FeedManager | `financial-api/app/realtime/` |
| Supply | crawler, dynamic scripts sandbox, arq, streaming worker | `app/crawler/`, `scripts_runtime/`, `worker/` |
| Admin | Vue 3 SPA, capabilities codegen | `financial-admin/` |

### Backend three-process model (highest-frequency trap)

| Process | Command | Purpose |
|---------|---------|---------|
| API | `uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000` | HTTP + WS entry |
| arq | `uv run arq worker.settings.WorkerSettings` | Short batch: crawl job, script run |
| streaming | `uv run python -m worker.streaming` | Long-lived WS crawler (separate process) |

**Full crawl/script loop needs all three processes running simultaneously.**

> **Never** run streaming tasks inside arq. They are architecturally separate.

### API response envelope (highest-frequency trap)

| | Financial | QuantDinger Human (DO NOT mix) |
|--|-----------|-------------------------------|
| Success `code` | **`0`** | `1` |
| Helpers | `success()` / `error()` / `paginated()` | Different |
| Prefix | `/api/<domain>/...` | `/api/...` |

### Root-level docs

| Path | Role |
|------|------|
| `financial/AGENTS.md` | Cross-project router (which sub-repo to open) |
| `financial/docs/rules/capability-activation.md` | Cross-repo capability normative |
| `financial/docs/reviews/README.md` | Guide chronicle index |
| `financial-fullstack-expert/` | **This expert pack** (workspace-level skill, tool-agnostic) |

### Authority order (financial tasks)

```text
1. Scenario routing / checklist / Touch Map → this expert pack
2. must / must-not → sub-repo docs/rules/** (+ root capability-activation)
3. Legacy how-to → docs/skills/project (only when pack does not cover)
4. Workspace rules → personal global skills (lowest)
```

## Do / Don't

| Do | Don't |
|----|-------|
| Treat each repo as independent Git | Don't commit cross-repo in one operation |
| Run `financial-api` .venv in Windows PowerShell (`uv sync` / `uv run`) | Don't use WSL for `uv sync` — Linux symlinks pollute Windows venv |
| Run server-side tests / bash in WSL Ubuntu | Don't use PowerShell `Set-Content` for file ops (BOM issues) |
| Check three-process model before touching crawler/worker | Don't assume single-process backend |
| Follow `code: 0` envelope | Don't mix QuantDinger `code: 1` |

## Touch map

| What | Path (repo-relative) |
|------|---------------------|
| Root agent entry | `financial/AGENTS.md` |
| Web agent entry | `financial-web/AGENTS.md` |
| API agent entry | `financial-api/AGENTS.md` |
| Admin agent entry | `financial-admin/AGENTS.md` |
| Web rules index | `financial-web/docs/rules/README.md` |
| API rules index | `financial-api/docs/rules/README.md` |
| Admin rules | `financial-admin/docs/rules/README.md` |
| Contracts SSOT | `financial-web/packages/contracts/` |
| Contract matrix | `financial-web/src/core/api/CONTRACT.md` |
| Cross-repo sync rule | `financial-api/docs/rules/core/web-integration.md` |
| Capability-activation | `financial/docs/rules/capability-activation.md` |
| Reviews index | `financial/docs/reviews/README.md` |

## Sequences

1. **Cold start:** Read `AGENTS.md` (root) → identify sub-repo → Read sub-repo
   `AGENTS.md` → Read this pack + `IDENTITY.md` + `ROUTER.md`.
2. **Cross-stack task:** Route via `ROUTER.md` → load pack 01 (contracts) +
   pack 02 or 03 (architecture) + pack 10 (playbook) → code → verify with
   pack 11.
3. **Admin task:** Read `financial-admin/docs/rules/conventions.md` + `page.md`
   → use pack 06 (RCC/capability) for cross-repo config.

## Verify

```bash
# Check repo paths exist (run from financial/ root)
# PowerShell:
@('financial-web/AGENTS.md','financial-api/AGENTS.md','financial-admin/AGENTS.md',
  'financial-web/packages/contracts/','financial-web/src/core/api/CONTRACT.md',
  'financial/docs/rules/capability-activation.md'
) | ForEach-Object { if (Test-Path $_) { "OK $_" } else { "MISSING $_" } }
```

## SSOT links

| Doc | Path |
|-----|------|
| Root AGENTS | `financial/AGENTS.md` |
| Capability-activation | `financial/docs/rules/capability-activation.md` |
| Web rules README | `financial-web/docs/rules/README.md` |
| API rules README | `financial-api/docs/rules/README.md` |
| CONTRACT matrix | `financial-web/src/core/api/CONTRACT.md` |
| Capability-activation | `financial/docs/rules/capability-activation.md` |

## Related packs

- **01** — Contracts SSOT (sync commands, domain matrix)
- **02** — Web architecture (layering, modules, barrel exports)
- **03** — API architecture (Bounded Context, layer model, pkgutil)
- **11** — Verification (commands for all three repos)
