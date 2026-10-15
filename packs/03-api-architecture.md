# Pack 03 — API Architecture

## When to load

- Adding a new FastAPI domain, endpoint, or service.
- Touching database models, migrations, seed data, or fixtures.
- Changing API layering, dependency injection, or response patterns.
- Debugging domain auto-discovery or import alias issues.

## Scope / Out of scope

**Scope:** Bounded Context table, layer model, pkgutil auto-discovery,
seed/fixtures touchpoints, response envelope, dependency injection, database
rules summary, no import alias.

**Out of scope:** Contract sync (→ pack 01), web frontend architecture (→ pack
02), realtime channel details (→ pack 05), verification commands
(→ pack 11).

## Must know

### Layer model (must obey)

```text
router → service → repository → models
                   → schemas (Pydantic v2)
                   → ports (optional, protocol interfaces)
```

Middleware chain (outer → inner):
```text
LoggingMiddleware → RequestIDMiddleware → RateLimitMiddleware → CORS
```

**Hard rules:**
- New features go in `app/domains/<context>/<name>/`.
- Domain routers are **auto-discovered** via `pkgutil.iter_modules` in
  `app/api/v1/router.py` — no manual registration.
- **No import aliases** — always use full module paths (e.g.
  `from app.repositories.market import snapshots`).
- Repository layer holds SQLAlchemy read queries only — no HTTP, no response
  shaping.
- Routes do not contain business logic; services orchestrate use cases.

### Bounded Contexts

| Context | Route prefix (main) | Sub-domain examples |
|---------|---------------------|---------------------|
| `market/` | `/api/market`, `/quote`, `/trade`, `/signals`, `/market-trends`, `/calendar` | snapshot, quote, trade, signals, brokers, market_trends, calendar |
| `content/` | `/api/news`, `/analysis`, `/ads`, `/data-center`, `/learning`, `/icons` | news, analysis, ads, data_center, learning, icons |
| `engagement/` | `/api/chat`, `/qa` | chat, qa |
| `system/` | `/api/home`, `/navigation`, `/logs`, ... | home, navigation, app_config, scripts, audit, debug, crawler_feed, datasource_config |
| `auth/` | `/api/auth` | login, register, email verification, MFA |

> Quotes / chat / qa / news-flash also go through **WebSocket** `/api/ws`.

### Adding a new domain

```text
app/domains/<context>/<name>/
  schemas.py      # Pydantic v2 request/response models
  router.py       # FastAPI router
  service.py      # business logic
  (optional) repository.py  # if DB read needed
```

Steps:
1. Create the directory with `schemas.py`, `router.py`, `service.py`.
2. `pkgutil` auto-discovers the router — no manual registration needed.
3. If DB read: add repository in `app/repositories/<context>/`.
4. If new table: design → Alembic migration → seed → repository → service.
5. Update `CONTRACT.md` domain matrix (pack 01).

### Response envelope (must use)

```python
from app.core.response import success, error, paginated

# Success
return success(data)                                        # { code: 0, msg, data }

# Error (raise AppError subclass)
raise NotFoundError("Item", identifier=42)                  # → global handler

# Pagination
return paginated(items, total=100, page=p.page, page_size=p.page_size)
# → { code: 0, msg, data: { list, total, page, pageSize } }
```

> Financial success = **`code: 0`**. Never use QuantDinger `code: 1`.

### Dependency injection

```python
from app.dependencies import DbSession, CurrentUser, Pagination
from app.core.response import success

@router.get("/items", response_model=ApiResponse[list[ItemRead]])
def list_items(db: DbSession, user: CurrentUser, p: Pagination = Depends(get_pagination)):
    return success(service.get_items(db, user.id, p))
```

### Exception handling

- Raise `AppError` subclasses (`NotFoundError`, `ValidationError`,
  `UnauthorizedError`, etc.) with `status_code`.
- Global handler in `main.py` converts to `{ code, msg, data }`.
- Unknown exceptions return 500 + stack trace logged.

### Database rules summary

- Target DB: `quant_zc` (local Docker or shared instance).
- Engine lazy-init via `@lru_cache`; `reset_engine()` for test config.
- Existing `qd_*` tables are **read-only**; new tables use `fin_` prefix.
- **Never** `Base.metadata.create_all()` on shared library; use Alembic.
- New tables: `__table_args__` must include `{"comment": "table purpose"}`;
  every `mapped_column` must have `comment="field purpose"`.
- Schema change flow: design → Alembic migration → seed → repository → service.

### Data type selection (JSONB vs row-level)

**Core principle:** JSONB is acceptable only when the value is read/written as a
**whole unit**. If Admin CRUD operates on individual elements inside a JSONB
array/object, that data MUST use row-level columns or a child table.

| Data pattern | Use | Example |
|---|---|---|
| Scalar | `String`/`Integer`/`Boolean`/`Float`/`DateTime` | Default for all single-value fields |
| Whole-unit object | `JSONB` | `{"enabled": true, "target": "site"}` — Admin edits entire form |
| List of scalars | `ARRAY(String)` | Tags, symbol code lists |
| List of structured objects (Admin per-item CRUD) | **Child table** | `fin_ad_placements` (was `ads.placement_catalog` JSONB) |
| Object keys Admin edits independently | **Independent columns** | `rail_group` / `nav_sort_order` (was `meta` JSONB) |
| Crawler snapshot (whole-unit read/write) | `JSONB` | `chart_points`, `pending_rows`, `candles` |

> Full decision tree: `financial-api/docs/rules/modules/database/data-type-selection.md`

### Seed data safety

- Fixture SSOT: `app/db/fixtures/` (loaded by `app/db/seeders/`).
- All seeders must be **idempotent** — never overwrite runtime data.
- User accounts use **ensure-exists** semantics: `seed_users` creates
  `admin`/`demo` when missing (default password `123456`, env-overridable
  with warning) but never touches existing rows; `SEED_RESET_PASSWORDS=1`
  explicitly opts in to password resets (demo/staging only).
- Seed entry: `uv run python -m app.db.seed`.
- Fixture export for MSW: `uv run python -m scripts.export_fixtures_json`.

### Config management

- Settings: mixin-based (`_base`, `_server`, `_database`, `_auth`, `_logging`,
  `_crawler`, `_vendors`).
- Env loading priority: `.env` (always) → `$ENV_FILE` (override) → process env.
- ⚠️ `.env` is read by pydantic Settings and is **not** exported into
  `os.environ`. Runtime code must read values via `get_settings()` —
  `os.environ.get("SOME_ENV_KEY")` silently returns `None` even when the
  key exists in `.env` (shipped as a real bug: the mock-candle switch
  never took effect).
- Never hardcode env vars, business rules, or feature flags.

### Logging

- Use `get_logger()` with dotted naming; **never** bare
  `logging.getLogger(__name__)`.
- Logger naming: `app.main`, `app.domains.<name>.service`, `realtime.<component>`,
  `crawler.<component>`, `middleware.<name>`, `auth.<component>`.
- Never `print()` for debugging. Never log sensitive data.

## Do / Don't

| Do | Don't |
|----|-------|
| Put new features in `domains/<context>/<name>/` | Don't add import aliases; use full module paths |
| Use `success(data)` with `code: 0` | Don't use QuantDinger `code: 1` |
| Raise `AppError` subclasses for errors | Don't return raw dicts or HTTPException |
| Run Alembic for schema changes | Don't use `Base.metadata.create_all()` on shared DB |
| Make seeders idempotent | Don't overwrite runtime data in seed |
| Use `get_logger()` with dotted naming | Don't use bare `logging.getLogger(__name__)` or `print()` |
| Add `comment` to every table and column | Don't leave ORM fields without comments |
| Use JSONB only for whole-unit read/write | Don't use JSONB for Admin per-item CRUD — use child tables or columns |

## Touch map

| What | Path |
|------|------|
| Domain routers auto-discovery | `financial-api/app/api/v1/router.py` |
| Domains root | `financial-api/app/domains/` |
| Market context | `financial-api/app/domains/market/` |
| Content context | `financial-api/app/domains/content/` |
| Engagement context | `financial-api/app/domains/engagement/` |
| System context | `financial-api/app/domains/system/` |
| Auth context | `financial-api/app/domains/auth/` |
| Repositories | `financial-api/app/repositories/` |
| Models | `financial-api/app/models/` |
| Schemas (per domain) | `financial-api/app/domains/<ctx>/<name>/schemas.py` |
| DI | `financial-api/app/dependencies.py` |
| Response helpers | `financial-api/app/core/response.py` (or `responses.py`) |
| Exceptions | `financial-api/app/core/exceptions.py` |
| Database engine | `financial-api/app/core/database.py` |
| Config | `financial-api/app/config/` |
| Middleware | `financial-api/app/middleware/` |
| Migrations | `financial-api/alembic/` or `financial-api/migrations/` |
| Fixtures SSOT | `financial-api/app/db/fixtures/` |
| Seeders | `financial-api/app/db/seeders/` |
| Seed entry | `financial-api/app/db/seed.py` |
| Architecture rule | `financial-api/docs/rules/core/architecture.md` |
| API conventions rule | `financial-api/docs/rules/core/api-conventions.md` |
| Domain design rule | `financial-api/docs/rules/core/domain-design-principles.md` |
| Database rule | `financial-api/docs/rules/modules/database/database.md` |
| Seed rule | `financial-api/docs/rules/modules/database/seed.md` |
| Table design rule | `financial-api/docs/rules/modules/database/table-design.md` |
| Logging rule | `financial-api/docs/rules/core/logging.md` |

## Sequences

1. **New domain endpoint:** Create `schemas.py` → `service.py` → `router.py`
   → (if DB) `repository.py` + model → run `pytest` → `pnpm api:export` +
   `api:sync` → update `CONTRACT.md`.
2. **New table:** Design table with comments → Alembic migration (`alembic
   revision --autogenerate`) → add to `models/` → create fixture → add seeder
   → `uv run python -m app.db.seed` → add repository → wire into service.
3. **New seed config:** Add fixture entry in `app/db/fixtures/config/app.py`
   → register ConfigDefinition in `registry.py` → run seed → verify
   `listConfigs(namespace)`.

## Verify

```bash
# From financial-api/
uv run ruff check .          # static analysis (lint)
uv run pytest -q             # unit + integration tests
uv run alembic upgrade head  # apply migrations
uv run python -m app.db.seed # load seed data
```

## SSOT links

| Doc | Path |
|-----|------|
| Architecture | `financial-api/docs/rules/core/architecture.md` |
| API conventions | `financial-api/docs/rules/core/api-conventions.md` |
| Domain design | `financial-api/docs/rules/core/domain-design-principles.md` |
| Database | `financial-api/docs/rules/modules/database/database.md` |
| Data type selection | `financial-api/docs/rules/modules/database/data-type-selection.md` |
| Seed safety | `financial-api/docs/rules/modules/database/seed.md` |
| Table design | `financial-api/docs/rules/modules/database/table-design.md` |
| Logging | `financial-api/docs/rules/core/logging.md` |
| Configuration | `financial-api/docs/rules/core/configuration.md` |
| API AGENTS | `financial-api/AGENTS.md` |

## Related packs

- **00** — Workspace map (repo layout, three-process model)
- **01** — Contracts SSOT (OpenAPI export/sync after endpoint changes)
- **02** — Web architecture (frontend `core/api` counterpart)
- **10** — Fullstack playbook (pair-update includes API side)
- **11** — Verification (`ruff` + `pytest` details)
