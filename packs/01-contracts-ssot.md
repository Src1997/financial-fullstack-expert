# Pack 01 — Contracts SSOT

## When to load

- Any task involving cross-stack data shape changes (API, navigation, realtime,
  capabilities, home blocks, labels).
- Before running or auditing any `pnpm sync:*` / `pnpm codegen:*` command.
- When adding a new API domain or changing an existing endpoint contract.

## Scope / Out of scope

**Scope:** Contract fan-out matrix, domain matrix, sync/codegen commands,
generated artifact ban, audit commands.

**Out of scope:** Deep API endpoint design (→ pack 03), web `core/api` module
shape (→ pack 02), verification command details (→ pack 11).

## Must know

### Contract fan-out matrix

> SSOT = author here; generated = never hand-edit.

| Contract | SSOT location | Sync / Codegen | Generates | Audit |
|----------|---------------|----------------|-----------|-------|
| Nav+Route manifest (primary) | `packages/contracts/navigation-manifest/` (`entries/`, `permissions.yaml`, `shell-fixed.yaml`, `manifest.lock.json`) | `pnpm codegen:navigation` / `pnpm nav:add` | `modules/navigation-platform/generated/*` (menuIds, permissions, registry, routes, obsolete); api `contract.py` markers | `pnpm codegen:navigation:check`; `pnpm deps:navigation-platform` |
| Realtime topics | `packages/contracts/realtime-topics.json` | `pnpm sync:realtime-topics` | web `modules/websocket/topic-registry.ts` (generated block); api `app/realtime/topic_registry.py` (generated block) | `pnpm deps:realtime-topics` |
| Admin capabilities | `packages/contracts/admin-capabilities.json` | `pnpm sync:admin-capabilities` | admin `src/shared/constants/admin-capabilities.ts`; api `app/domains/system/app_config/router.py` VALID_CAPABILITIES | `pnpm deps:contracts` |
| OpenAPI types | `packages/contracts/openapi.json` | `pnpm api:export` + `pnpm api:sync` | `financial-web/src/types/api/generated/schema.d.ts` | `pnpm api:sync:check` |
| Home block kinds | `packages/contracts/home-block-kinds.json` | `pnpm codegen:home-block-kinds` | web `HOME_BLOCK_KINDS` (page-layout.ts); api registry.py `HOME_BLOCK_KINDS` | `pnpm codegen:home-block-kinds:check` |
| C-end nav labels | web `locales/modules/system/navigation/zh-CN.ts` + `mega-menu/zh-CN.ts` | `pnpm codegen:c-end-nav-labels` | admin `src/shared/i18n/cEndNavLabels.ts` | `pnpm codegen:c-end-nav-labels:check` |
| Nav permissions (legacy) | `packages/contracts/navigation-permissions.json` | `pnpm sync:navigation-permissions` | featureFlags / menuPermissions parity artifacts | `pnpm deps:navigation-permissions` |

> Legacy nav-permissions is read-only backup during manifest migration. Prefer
> Manifest codegen. Legacy sync remains only for parity audits.

### API domain matrix (from CONTRACT.md)

| Domain (`core/api`) | HTTP prefix | Backend router | Status |
|---------------------|-------------|---------------|--------|
| `auth` | `/auth` | Yes | Aligned |
| `navigation` | `/navigation` | Yes | Aligned |
| `home` | `/home` | Yes | Aligned |
| `market` | `/market` | Yes | Aligned |
| `quote` | `/quote` | Yes | Aligned |
| `news` | `/news` | Yes | Aligned |
| `calendar` | `/calendar` | Yes | Aligned |
| `qa` | `/qa` | Yes | Aligned |
| `chat` | `/chat` | Yes | Aligned |
| `trade` | `/trade` | Yes | Aligned |
| `ads` | `/ads` | Yes | Aligned |
| `icons` | `/icons` | Yes | Aligned |
| `learning` | `/learning` | Yes | Aligned |
| `market-trends` | `/market-trends` | Yes | Aligned |
| `data-center` | `/data-center` | Yes | Aligned |
| `analysis` | `/analysis` | Yes | Aligned |
| `signals` | `/signals` | Yes | Aligned |

> No backend domain lacks a frontend `core/api` module today (except health
> `/health`).

### Adding a new domain

1. Add row to `CONTRACT.md` domain matrix.
2. Create `financial-web/src/core/api/<domain>/` with `types.ts` + `index.ts`
   (+ optional `mock.ts`).
3. Add `export * from './<domain>'` to `src/core/api/index.ts`.
4. Add backend `app/domains/<context>/<name>/router.py` + seed.
5. Run `pnpm api:sync` when backend contract changes.

### Home dashboard vs domain APIs

- Home page blocks use **per-section loaders**
  (`domains/home/config/section-sources.ts`).
- Each `HomePageSection` fetches on viewport entry (`useHomeSectionViewport`).
- `GET /home/dashboard` is **deprecated for new home UI** (kept for external
  consumers). Use granular domain APIs.

## Do / Don't

| Do | Don't |
|----|-------|
| Change SSOT JSON/YAML first, then run sync/codegen | Don't hand-edit `generated/*`, `schema.d.ts`, or generated topic-registry blocks |
| Run the matching `:check` command after sync | Don't skip audit — `pnpm check` will catch drift |
| Update `CONTRACT.md` when adding/changing a domain | Don't leave the matrix stale |
| Run `pnpm api:export` from backend before `pnpm api:sync` | Don't run `api:sync` without first exporting if backend changed |
| Prefer Manifest codegen for nav changes | Don't use legacy `navigation-permissions.json` for new work |

## Touch map

| What | Path |
|------|------|
| Contracts root | `financial-web/packages/contracts/` |
| OpenAPI SSOT | `financial-web/packages/contracts/openapi.json` |
| Nav manifest SSOT | `financial-web/packages/contracts/navigation-manifest/` |
| Realtime topics SSOT | `financial-web/packages/contracts/realtime-topics.json` |
| Admin caps SSOT | `financial-web/packages/contracts/admin-capabilities.json` |
| Home block kinds SSOT | `financial-web/packages/contracts/home-block-kinds.json` |
| Domain matrix | `financial-web/src/core/api/CONTRACT.md` |
| Generated OpenAPI types | `financial-web/src/types/api/generated/schema.d.ts` |
| Generated nav artifacts | `financial-web/src/modules/navigation-platform/generated/` |
| Generated topic registry (web) | `financial-web/src/modules/websocket/topic-registry.ts` |
| Generated topic registry (api) | `financial-api/app/realtime/topic_registry.py` |
| Generated admin caps (admin) | `financial-admin/src/shared/constants/admin-capabilities.ts` |
| API capabilities patch | `financial-api/app/domains/system/app_config/router.py` |
| Cross-repo sync rule | `financial-api/docs/rules/core/web-integration.md` |
| Contracts README | `financial-web/packages/contracts/README.md` |

## Sequences

1. **OpenAPI change:** Edit backend router → `pnpm api:export` (from
   `financial-web`) → `pnpm api:sync` → `pnpm api:sync:check` → update
   `core/api` module if shape changed.
2. **Nav manifest change:** Edit manifest YAML (or `pnpm nav:add`) →
   `pnpm codegen:navigation` → `pnpm codegen:navigation:check` → verify shell
   + routes.
3. **Realtime topic change:** Edit `realtime-topics.json` →
   `pnpm sync:realtime-topics` → `pnpm deps:realtime-topics` → verify web +
   api registries match.

## Verify

```bash
# All commands run from financial-web/ unless noted
pnpm api:sync:check           # OpenAPI type drift check
pnpm codegen:navigation:check # Nav manifest drift check
pnpm deps:realtime-topics     # Realtime topic parity audit
pnpm deps:contracts           # Admin capabilities + contract audit
pnpm codegen:home-block-kinds:check
pnpm codegen:c-end-nav-labels:check
pnpm deps:navigation-permissions  # Legacy parity (migration period)

# Full quality gate (includes contract checks):
pnpm check
```

## SSOT links

| Doc | Path |
|-----|------|
| CONTRACT.md | `financial-web/src/core/api/CONTRACT.md` |
| Cross-repo sync rule | `financial-api/docs/rules/core/web-integration.md` |
| Contracts README | `financial-web/packages/contracts/README.md` |
| Web api-conventions | `financial-web/docs/rules/api-conventions.md` |
| API api-conventions | `financial-api/docs/rules/core/api-conventions.md` |
| Root AGENTS contracts table | `financial/AGENTS.md` § "Contracts & Sync" |

## Related packs

- **00** — Workspace map (repo boundaries)
- **03** — API architecture (domain design, how to add a new domain)
- **10** — Fullstack playbook (pair-update checklist includes contract step)
- **11** — Verification (full command matrix)
