# Pack 11 — Verification

## When to load

- **Always** — load for any code change to know how to verify.
- Before claiming work is "done" or "fixed".
- Before creating a PR.
- When debugging a test or check failure.

## Scope / Out of scope

**Scope:** Verification commands for all three repos, quality check matrix,
shell split guidance, pre-PR checklist.

**Out of scope:** CI/CD pipeline setup (DevOps), deployment verification.

## Must know

### Verification matrix

| Change type | Web | API | Contract | Manual |
|-------------|-----|-----|----------|--------|
| REST domain | `core/api` + page lint | `pytest` + `ruff` | `api:sync` | Login → exercise path |
| Manifest menu | `codegen:navigation` + shell | navigation seed | `codegen:navigation:check` | Three-role menu check |
| Realtime | `topic-registry.ts` | channels + hub | `sync:realtime-topics` | Subscribe + push language |
| RCC | `runtime-config` getters | `app_config` | `sync:admin-capabilities` (if caps) | Admin change → C-end reflects |
| Crawler / script | — | worker three-process | — | Enqueue once |
| Style / theme | tokens / theme | — | — | Switch theme/trend |
| Docs expert | `deps:rules-docs` | — | — | Shell check S1–S6 |

### Web commands (financial-web)

All commands run from `financial-web/`:

```bash
# Full quality gate — includes type + lint + format + unit + architecture + contract + MSW coverage
pnpm check

# Individual checks
pnpm deps:boundaries           # circular dependency + module boundary
pnpm deps:rules-docs           # rules doc link audit
pnpm deps:commenting           # cross-repo comment marker audit
pnpm deps:contracts            # admin capabilities + contract audit
pnpm deps:realtime-topics      # realtime topic parity audit
pnpm deps:mock-coverage        # MSW mock coverage audit

# Codegen drift checks
pnpm api:sync:check            # OpenAPI type drift
pnpm codegen:navigation:check  # Nav manifest drift
pnpm codegen:home-block-kinds:check
pnpm codegen:c-end-nav-labels:check

# Build
pnpm build:prod                # production build (runs api:sync:check first)
```

### API commands (financial-api)

All commands run from `financial-api/`:

```bash
# Static analysis (lint)
uv run ruff check .

# Unit + integration tests
uv run pytest -q

# Database
uv run alembic upgrade head     # apply migrations
uv run python -m app.db.seed    # load seed data

# OpenAPI export (from financial-web/)
# pnpm api:export   # export from backend → packages/contracts/openapi.json

# Multi-worker smoke (local Redis + multi-worker)
uv run python scripts/multiworker_smoke.py

# Fixture export for MSW
uv run python -m scripts.export_fixtures_json

# Crawler dry-run
uv run python -m app.crawler run flash_news --dry-run
```

### Admin commands (financial-admin)

All commands run from `financial-admin/`:

```bash
pnpm dev          # dev server (:5174)
pnpm test         # unit tests
pnpm type-check   # TypeScript type check
pnpm build        # production build
```

### Shell split (workspace convention)

| Task | Shell | Why |
|------|-------|-----|
| Server-side tests, bash scripts, deploy verification, LF handling | WSL Ubuntu | Avoids CRLF / `$` / quoting issues |
| `uv sync` / `uv run` for financial-api | Windows PowerShell | WSL creates Linux symlinks that pollute Windows venv |
| Packaging scripts (`build.ps1`, `pack-*.ps1`) | Windows PowerShell | Windows-native tooling |
| Windows-side upload (`scp`) | Windows PowerShell | Windows path compatibility |
| Frontend `pnpm` commands | Either (WSL preferred) | Both work; WSL for consistency |

**WSL format:**
```bash
wsl bash -c "cd /mnt/d/Workspace/financial/financial-api && uv run pytest -q"
```

> **Never** use PowerShell `Set-Content` / `Where-Object` for file operations —
> they add BOM, break Chinese encoding, and corrupt JSON structure.

## Do / Don't

| Do | Don't |
|----|-------|
| Run `pnpm check` before claiming web work done | Don't skip verification to save time |
| Run `uv run ruff check .` + `uv run pytest -q` for API changes | Don't only run one side of a cross-stack change |
| Run `pnpm api:sync:check` after backend contract changes | Don't commit stale `schema.d.ts` or drift |
| Use WSL for server-side tests | Don't use PowerShell for pytest (CRLF issues) |
| Run `pnpm build:prod` for structural web changes | Don't rely on `pnpm dev` alone (it skips some errors) |
| Export OpenAPI after route changes (`pnpm api:export`) | Don't commit stale `openapi.json` |

## Touch map

| What | Path |
|------|------|
| Web package.json (all scripts) | `financial-web/package.json` |
| API pyproject.toml (ruff config) | `financial-api/pyproject.toml` |
| Admin package.json | `financial-admin/package.json` |
| Web CONTRACT.md | `financial-web/src/core/api/CONTRACT.md` |
| API web-integration rule | `financial-api/docs/rules/core/web-integration.md` |
| Web rules README (verification section) | `financial-web/docs/rules/README.md` |
| API rules README (verification section) | `financial-api/docs/rules/README.md` |
| Contracts README | `financial-web/packages/contracts/README.md` |

## Sequences

1. **Pre-PR verification:** Run web `pnpm check` → API `ruff` + `pytest` →
   contract `api:sync:check` → manual exercise.
2. **Contract drift debugging:** Run `pnpm api:sync:check` → if drift,
   `pnpm api:export` (from web, reads backend) → `pnpm api:sync` → re-check.
3. **Test failure debugging:** Load pack 11 + relevant architecture pack (02
   or 03) + check the specific test file path.

## Verify

```bash
# === Web (from financial-web/) ===
pnpm check

# === API (from financial-api/) ===
uv run ruff check .
uv run pytest -q

# === Admin (from financial-admin/) ===
pnpm type-check

# === Contract sync (from financial-web/) ===
pnpm api:sync:check
pnpm deps:contracts
pnpm deps:realtime-topics

# === Expert pack shell check (from financial/) ===
# PowerShell:
@(
  'IDENTITY.md','ROUTER.md',
  'packs/00-workspace-map.md',
  'packs/01-contracts-ssot.md',
  'packs/02-web-architecture.md',
  'packs/03-api-architecture.md',
  'packs/10-fullstack-playbook.md',
  'packs/11-verification.md',
  'adapters/generic-system-prompt.md',
  'adapters/cursor.md'
) | ForEach-Object { if (-not (Test-Path $_)) { throw "MISSING $_" } else { "OK $_" } }
```

## Pre-PR checklist

### Web

- [ ] `pnpm check` passes (type + lint + format + unit + architecture +
      contract + MSW coverage)
- [ ] `pnpm deps:boundaries` passes — no shell→domains, no shared→core/api, no shared→domains
- [ ] All user-visible text uses i18n (no hardcoded Chinese/English)
- [ ] No `any` without explicit justification
- [ ] API endpoint strings in `core/api` modules only (not in pages/components)
- [ ] No deep private imports; use barrel exports
- [ ] Scoped SCSS uses `@use` + CSS variables (no `@import`)
- [ ] Shell (`src/shell/`) does not `import @/domains/**` (red line #12)
- [ ] Shared (`src/shared/`) does not `import @/core/api` or `@/domains/**` (red line #13)

### API

- [ ] `uv run ruff check .` passes
- [ ] `uv run pytest -q` passes (or relevant subset)
- [ ] No `print()` debug statements (use `get_logger()`)
- [ ] No bare `logging.getLogger(__name__)`; use `get_logger()` with dotted name
- [ ] No secrets / `.env` in diff
- [ ] New tables have `comment` on `__table_args__` + every `mapped_column`
- [ ] Seeders are idempotent
- [ ] Production `lifespan` does not auto-run `alembic upgrade` / `run_seed` (red line #14)
- [ ] `admin.*` / `navigation.*` / role-matrix ConfigDefinitions are `is_public=False` (red line #15)
- [ ] Control-plane write paths call `audit_log()` in the same transaction
- [ ] Four Admin control-plane tables exist: `fin_admin_role_capabilities`, `fin_config_revisions`, `fin_role_permission_bindings`, `fin_admin_sessions` (red line #16)
- [ ] All CMS tables inherit `TimestampMixin`; table names pluralized (red line #17)
- [ ] `seed_users` has no hardcoded weak passwords; `SKIP_SEED_PURGE` default-on in prod (red line #18)

### Cross-stack

- [ ] Backend response shape matches frontend expectation (`code: 0`)
- [ ] `core/api` module updated if endpoint paths/shapes changed
- [ ] `openapi.json` updated if contract changed (`pnpm api:export` + `api:sync`)
- [ ] `CONTRACT.md` domain matrix updated if new domain
- [ ] i18n keys added for any new user-visible text
- [ ] Field naming: camelCase across front/back
- [ ] Time format: ISO 8601

### Expert pack (if this pack changed)

- [ ] Shell check S1–S6 passed (see acceptance doc)
- [ ] CHANGELOG.md updated
- [ ] ROUTER.md scenario table still covers relevant rows

## SSOT links

| Doc | Path |
|-----|------|
| Web rules README (verification) | `financial-web/docs/rules/README.md` § Verification |
| API rules README (verification) | `financial-api/docs/rules/README.md` § Verification |
| Cross-repo sync rule | `financial-api/docs/rules/core/web-integration.md` |
| Contracts README | `financial-web/packages/contracts/README.md` |
| Web AGENTS | `financial-web/AGENTS.md` |
| API AGENTS | `financial-api/AGENTS.md` |
| Root AGENTS (commands) | `financial/AGENTS.md` § Common Commands |

## Related packs

- **00** — Workspace map (where things are)
- **01** — Contracts SSOT (sync/check commands context)
- **02** — Web architecture (`pnpm check` context)
- **03** — API architecture (`ruff` + `pytest` context)
- **10** — Fullstack playbook (verification matrix as final step)
