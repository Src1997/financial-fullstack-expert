# Pack 10 — Fullstack Playbook

## When to load

- Building an end-to-end new feature that spans backend + frontend.
- Any change that touches both `financial-api` and `financial-web` in the same
  task.
- When debugging "I changed the backend but the frontend doesn't reflect it"
  (or vice versa).

## Scope / Out of scope

**Scope:** End-to-end pair-update checklist, domain classification (A–H),
sequence from contract to verification, impact assessment.

**Out of scope:** Deep contract commands (→ pack 01), web architecture details
(→ pack 02), API architecture details (→ pack 03), verification command list
(→ pack 11).

## Must know

### Domain classification (from scheme §3)

| Domain | Typical task | Primary packs |
|--------|-------------|---------------|
| A — Contracts | sync OpenAPI / topics / caps / manifest | 01, 10, 11 |
| B — Web platform | new module, page, component, style | 02, 10 |
| C — API domains | new endpoint, table, seed | 03, 10, 11 |
| D — Navigation / Chrome | menu, shell, route, home block | 04, 02 |
| E — Realtime + i18n | WS channel, push language, locale | 05, 02 |
| F — RCC / Capability | runtime config, admin config, caps | 06, 01 |
| G — Supply | crawler, scripts, arq, streaming | 07, 11 |
| H — Auth / Audit / Quality | login, guard, RBAC, audit | 08, 11 |

### Response envelope reminder

Financial success = **`code: 0`**. Frontend interceptor checks
`API_SUCCESS_CODE === 0`. Field naming: **camelCase** across front/back. Time:
ISO 8601 (datetime auto-serialized).

### Home page pattern

- Prefer **per-section loaders** (`domains/home/config/section-sources.ts`).
- Each `HomePageSection` fetches on viewport entry.
- **Do not** re-introduce a single `GET /home/dashboard` fetch in `HomePage`
  while section loaders exist.

## Do / Don't

| Do | Don't |
|----|-------|
| Define API contract (OpenAPI / SSOT) first | Don't start coding frontend before the contract is decided |
| Update both sides in one atomic change set | Don't change only the backend and forget `core/api` |
| Add i18n keys for all new user-visible text | Don't hardcode Chinese/English in components |
| Run `pnpm check` + `uv run ruff` + `uv run pytest` before claiming done | Don't skip verification |
| Evaluate realtime/worker impact | Don't assume a REST-only change has no WS implications |
| Use camelCase field names across front/back | Don't mix snake_case API responses with camelCase frontend |

## Touch map

| Step | What | Path |
|------|------|------|
| 1. Contract | OpenAPI SSOT | `financial-web/packages/contracts/openapi.json` |
| 1. Contract | Nav manifest SSOT | `financial-web/packages/contracts/navigation-manifest/` |
| 1. Contract | Realtime topics SSOT | `financial-web/packages/contracts/realtime-topics.json` |
| 1. Contract | Admin caps SSOT | `financial-web/packages/contracts/admin-capabilities.json` |
| 2. API | Domain router + service + schemas | `financial-api/app/domains/<ctx>/<name>/` |
| 2. API | Repository (if DB read) | `financial-api/app/repositories/<context>/` |
| 2. API | Model + migration | `financial-api/app/models/`, `financial-api/alembic/` |
| 2. API | Seed fixture | `financial-api/app/db/fixtures/` |
| 3. Frontend | core/api module | `financial-web/src/core/api/<domain>/` |
| 3. Frontend | Types (generated) | `financial-web/src/types/api/generated/schema.d.ts` |
| 3. Frontend | Page / component | `financial-web/src/domains/<name>/` |
| 3. Frontend | i18n | `financial-web/src/locales/modules/<group>/<name>/` |
| 4. RCC | Config fixture (if admin-tunable) | `financial-api/app/db/fixtures/config/app.py` |
| 4. RCC | Config registry | `financial-api/app/domains/system/app_config/registry.py` |
| 4. RCC | Web runtime-config getter | `financial-web/src/modules/runtime-config/` |
| 5. Verify | Web checks | `financial-web/` → `pnpm check` |
| 5. Verify | API checks | `financial-api/` → `uv run ruff check .` + `uv run pytest -q` |
| 5. Verify | Contract sync | `financial-web/` → `pnpm api:sync:check` |

## Sequences

### End-to-end new feature checklist

- [ ] **1. Classify domain** (A–H) and determine if cross-repo.
- [ ] **2. Update contract** (if API shape / menu / topics / caps / home-block
      involved):
  - [ ] Edit SSOT (openapi.json / manifest YAML / realtime-topics.json /
        admin-capabilities.json / home-block-kinds.json).
  - [ ] Run sync/codegen + `:check`.
  - [ ] Update `CONTRACT.md` domain matrix if new domain.
- [ ] **3. Backend** (if API change):
  - [ ] Create/update `schemas.py` → `service.py` → `router.py`.
  - [ ] If new table: design + comment → Alembic migration → model → fixture
        → seeder → `uv run python -m app.db.seed`.
  - [ ] Add/update repository if DB read.
  - [ ] Write/update tests.
  - [ ] `uv run ruff check .` + `uv run pytest -q`.
  - [ ] `pnpm api:export` (from `financial-web`).
- [ ] **4. Frontend**:
  - [ ] `pnpm api:sync` → update `core/api/<domain>/` (types + index + mock
        if needed).
  - [ ] Create/update page/component in `src/domains/<name>/`.
  - [ ] Add i18n keys (`zh-CN.ts` + `en-US.ts`).
  - [ ] Add path constants to `routerPaths.ts` if new route.
  - [ ] Use `el-skeleton` for loading, scoped SCSS for styles.
- [ ] **5. RCC / Admin** (if Feature-L / Platform-L config):
  - [ ] Add config fixture in `app/db/fixtures/config/app.py`.
  - [ ] Register `ConfigDefinition` in `registry.py`.
  - [ ] Add web `runtime-config` getter if needed.
  - [ ] Run `pnpm sync:admin-capabilities` if caps involved.
- [ ] **6. Realtime / Worker impact assessment**:
  - [ ] Does this change WS channels? → update topics + sync (pack 01 / pack 05).
  - [ ] Does this add a background task? → determine arq vs streaming (pack 07).
- [ ] **7. Verification matrix** (see pack 11):
  - [ ] `pnpm check` (web).
  - [ ] `uv run ruff check .` + `uv run pytest -q` (api).
  - [ ] `pnpm api:sync:check` (contract).
  - [ ] Manual: login → exercise path → verify both sides.
- [ ] **8. Touch Map audit**: if new files were created, check if pack Touch
      Maps need updating.

### "Why only one side changed" debugging

1. Load pack 01 + 10 + 11.
2. Check: was `pnpm api:sync` run after backend change?
3. Check: was `core/api` module updated to match new types?
4. Check: does frontend interceptor still use `code === 0`?
5. Check: is the route/path correct in `routerPaths.ts`?

## Verify

```bash
# Web (from financial-web/)
pnpm check                   # full quality gate

# API (from financial-api/)
uv run ruff check .
uv run pytest -q

# Contract sync (from financial-web/)
pnpm api:sync:check
```

## SSOT links

| Doc | Path |
|-----|------|
| CONTRACT.md | `financial-web/src/core/api/CONTRACT.md` |
| Root AGENTS | `financial/AGENTS.md` |
| Cross-repo sync rule | `financial-api/docs/rules/core/web-integration.md` |
| Web api-conventions | `financial-web/docs/rules/api-conventions.md` |
| API api-conventions | `financial-api/docs/rules/core/api-conventions.md` |
| Capability-activation | `financial/docs/rules/capability-activation.md` |

## Related packs

- **01** — Contracts SSOT (sync commands)
- **02** — Web architecture (`core/api` module shape)
- **03** — API architecture (domain design)
- **11** — Verification (full command matrix)
