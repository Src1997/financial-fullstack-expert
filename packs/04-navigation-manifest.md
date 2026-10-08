# Pack 04 — Navigation + Manifest

## When to load

- Adding, modifying, or deleting a C-end menu item, sidebar panel, or mega menu.
- Debugging "menu never visible", "panel blank", or permission parity failures.
- Changing feature flags or role-menu bindings.

## Scope / Out of scope

**Scope:** Menu SSOT + two sync tracks (Manifest / permissions parity), ownership
layers, add/modify/delete checklists, common failure table.

**Out of scope:** Contract sync commands in depth (→ pack 01), shell chrome
behavior (→ pack 02), backend seed details (→ api `modules/navigation.md`).

## Must know

### Ownership

| Layer | Path (financial-web/ unless noted) | Role |
| ----- | ---------------------------------- | ---- |
| Platform | `src/modules/navigation-platform/` | registry, resolver, policies, store, providers |
| Adapters | `src/shell/navigation-adapters/` | rail/header/mobile adapters, `panelRegistry`, mega-menu |
| Bootstrap | `src/app/bootstrap/navigation/setupNavigationPlatform.ts` | provider wiring |
| API | `src/core/api/navigation/` | fetch navigation menu |
| Route constants | `src/shared/constants/routerPaths.ts` | path SSOT |
| Backend | `financial-api/.../navigation/` + seed fixtures | menu + permission endpoints |

### Two sync tracks

| Track | SSOT | Command | Generates |
| ----- | ---- | ------- | --------- |
| Menu structure (Manifest) | `packages/contracts/navigation-manifest/` | `pnpm nav:add` / `pnpm codegen:navigation` | `modules/navigation-platform/generated/*`, api `contract.py` markers |
| Permission parity (legacy, parity-audited) | `packages/contracts/navigation-permissions.json` | `pnpm sync:navigation-permissions` | `menuPermissions.ts`, `featureFlags.ts`, `roles.ts` (web), `navigation-permissions.ts` (admin), `contract.py` (api) |

### Core rules

- Only **2 manual edits** when adding/removing a menu: the SSOT JSON +
  `menuIds.ts`. Everything else is generated or validated.
- Keep `id` aligned across registry, route constants, panel registry, and
  permission maps. Use path constants, never raw route strings.
- Do not hardcode menus in shell layout templates.
- Frontend policy may only **narrow** backend `allowedMenuIds`, never expand.

### Add a new menu item (checklist)

1. Contract SSOT: add to `allMenuIds` / `menuPermissionMap` /
   `defaultFeatureFlags` in `navigation-permissions.json` (new permission code
   → also `rolePermissions` per role).
2. Add constant to `src/modules/navigation-platform/constants/menuIds.ts`;
   run `pnpm sync:navigation-permissions` (auto-generates 5 files + validates
   api seed).
3. Registry + UI: node in `navigation-platform/registry/index.ts` (`kind`,
   `slots`, `meta.sortOrder`, `label`, `icon`); sidebar panel `*Sidebar.vue`
   under `src/domains/<name>/sidebar/` (auto-discovered); mega menu config in
   `shell/navigation-adapters/mega-menu/configs/` if needed; i18n keys.
4. Backend: navigation node fixture in
   `app/db/fixtures/navigation/navigation_nodes.py` + feature flag default in
   `app/db/fixtures/config/app.py` → `uv run python -m app.db.seed`.
5. Verify: `pnpm codegen:navigation:check` + `pnpm sync:navigation-permissions
   --check` + `pnpm deps:navigation-permissions` + `pnpm deps:navigation-platform`
   + `pnpm test:unit -- src/modules/navigation-platform src/shell/navigation-adapters`.

### Delete / deprecate

- Remove registry node + `menuIds.ts` constant + SSOT JSON entries → run sync →
  remove sidebar `.vue` / mega menu config / i18n keys.
- Backend: add the ID to `obsolete_node_ids` in `app/db/seeders/mock.py`,
  remove feature flag from fixtures, re-seed (seeder deletes obsolete rows).
- Deleted menu reappearing after seed = `obsolete_node_ids` not updated.

## Do / Don't

| Do | Don't |
|----|-------|
| Edit SSOT JSON + `menuIds.ts` only, then run sync | Don't hand-edit generated permission/flag files |
| Add sidebar panels as files (auto-discovered) | Don't manually register panels |
| Run `:check` / `deps:*` audits after sync | Don't trust a sync you didn't verify |
| Use `routerPaths.ts` constants | Don't hardcode route strings or menus in shell templates |

## Common failures

| Symptom | Cause |
| ------- | ----- |
| Rail icon visible but panel blank | Missing panel mapping in `panelRegistry` |
| Menu item never visible | Missing permission id / flag off / device mismatch |
| Mega menu section missing | `megaMenuRef` not registered |
| CI permission mismatch | SSOT updated but sync not run |
| Raw i18n key shown as label | Residual `fin_navigation_nodes` row merged via `mergeNavigationTrees` |
| Sync fails | `menuIds.ts` or SSOT JSON not updated first |

## Touch map

| What | Path |
| ---- | ---- |
| Manifest SSOT | `financial-web/packages/contracts/navigation-manifest/` |
| Permissions SSOT | `financial-web/packages/contracts/navigation-permissions.json` |
| Registry | `financial-web/src/modules/navigation-platform/registry/index.ts` |
| Panel registry | `financial-web/src/shell/navigation-adapters/panelRegistry.ts` |
| Path constants | `financial-web/src/shared/constants/routerPaths.ts` |
| Nav seed fixture | `financial-api/app/db/fixtures/navigation/navigation_nodes.py` |
| Nav rule (web) | `financial-web/docs/rules/navigation-platform.md` |
| Nav rule (api) | `financial-api/docs/rules/modules/navigation.md` |

## SSOT links

| Doc | Path |
| --- | ---- |
| Web navigation rule | `financial-web/docs/rules/navigation-platform.md` |
| Router-messenger skill | `financial-web/docs/skills/project/financial-web-router-messenger/` |
| Backend nav rule | `financial-api/docs/rules/modules/navigation.md` |
| Contract matrix (pack 01) | `financial-web/src/core/api/CONTRACT.md` |

## Related packs

- **01** — Contracts SSOT (Manifest vs permissions-parity tracks)
- **02** — Web architecture (shell / adapters layering)
