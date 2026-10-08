# Pack 02 — Web Architecture

## When to load

- Any task touching `financial-web` structure: modules, shared, bootstrap,
  pages, components, stores, styles.
- When adding a new module, page, shared component, composable, or SCSS.
- When debugging import boundary violations or auto-discovery issues.

## Scope / Out of scope

**Scope:** Web layering, `@/core/api` boundary, 22 modules inventory, barrel
exports, auto-discovery, i18n summary, TypeScript non-negotiables.

**Out of scope:** Contracts/sync (→ pack 01), API backend design (→ pack 03),
SCSS/tokens/theme details (→ pack 09), verification commands
(→ pack 11).

## Must know

### Directory structure

```text
financial-web/src/
  app/bootstrap/     # Startup wiring + provider bridges (only cross-layer bridge)
  core/              # api, config, router, stores (global core)
  modules/           # 22 platform capability modules
  shell/             # App shell: layout, sidebar, chrome, navigation-adapters
  domains/           # Business domains: pages, sidebar, components, composables, config
  shared/
    ui/              # Cross-domain UI: common / messaging / widgets
    lib/             # Cross-domain helpers and composables
    placeholders/    # Placeholder components
  styles/            # SCSS 7-layer system, main.scss entry
  locales/           # i18n modules (auto-discovered)
  types/             # Type definitions (incl. OpenAPI generated)
```

### Data flow (must obey)

```text
Domain page/component
  → @/core/api                         # sole business HTTP exit family
    → mockEnv.enabled ? mock.ts : @/modules/request  (inside API module only)
```

**Hard rules:**
- Consumers **never** `import @/modules/request` directly.
- Consumers **never** import `src/mock`, `core/api/**/mock`, or deep private
  files.
- Endpoint strings live inside API modules, never in pages/components.
- API barrel (`src/core/api/index.ts`) uses `export *` — adding a module = one
  line: `export * from './<domain>'`.

### API module shape

```text
src/core/api/<domain>/
  types.ts    # request/response types
  mock.ts     # MSW mock (private to module)
  index.ts    # public exports
```

### 22 platform modules

| Module | Grade | Frequency |
|--------|-------|-----------|
| `navigation-platform` | M | High |
| `runtime-config` | M | High |
| `websocket` | L | High |
| `i18n` | M | High |
| `theme` | M | High |
| `request` | L | High (API module internal only) |
| `icons` | L | Medium |
| `log` | M | Medium |
| `directives` | L | Medium |
| `motion` | L | Medium |
| `loading` | L/M | Medium |
| `notification` | M | Medium |
| `error` | L/M | Medium |
| `context-menu` | L | Medium |
| `cache` | S/M | Low–Medium |
| `storage` | M | Low–Medium |
| `toolkit` | M | Low–Medium |
| `hooks` | L | Low–Medium |
| `form` | L | Low–Medium |
| `export` | L | Low |
| `messaging` | S/M | Low |
| `tracking` | S/M | Low |

### Barrel exports (non-negotiable)

| Need | Import from |
|------|-------------|
| API | `@/core/api` |
| Router constants/nav | `@/core/router`, `@/core/router-messenger` |
| Config | `@/core/config` |
| Store | `@/core/stores` |
| Shell chrome | `@/shell/chrome` or `@/shell` |
| Shell composables | `@/shell/composables` |
| Shared UI | `@/shared/ui` |
| Shared helpers/composables | `@/shared/lib` |
| Platform tools | `@/modules/<name>` |

### Auto-discovery (no manual barrel registration)

| Feature | Mechanism |
|---------|-----------|
| Route pages | `import.meta.glob` in `core/router/modules/index.ts` |
| Sidebar panels | `import.meta.glob` in `shell/navigation-adapters/panelRegistry.ts` |
| Shared components | `unplugin-vue-components` global registration |
| i18n modules | `import.meta.glob('./*/*/index.ts')` in `locales/modules/index.ts` |

### i18n summary (non-negotiable)

- All user-visible text must use i18n. No hardcoded Chinese or English.
- Three contexts:
  - `$t('key')` in `<template>`
  - `useI18n().t` in `<script setup>`
  - `import { t } from '@/modules/i18n'` only in pure `.ts` (never in `.vue`,
    never top-level)
- Module structure: `src/locales/modules/<group>/<name>/` → `zh-CN.ts` +
  `en-US.ts` + `index.ts` (exports `zhCN` + `enUS`).

### TypeScript non-negotiables

- No `any` without explicit justification.
- Composables (`useXxx`) must define explicit return type interfaces (prevents
  TS2883/TS7056).
- Interfaces used by exported variables must be exported themselves (prevents
  TS4023).
- New domain terms must be added to `cspell.json`.

## Do / Don't

| Do | Don't |
|----|-------|
| Import via barrel exports (`@/core/api`, `@/shared/ui`) | Don't deep-import private files |
| Put cross-domain reusable UI in `src/shared/ui` | Don't duplicate UI in domain folders |
| Keep domain code in `src/domains/<name>/` | Don't let shell import route pages or domain internals |
| Use `<style lang="scss" scoped>` for component styles | Don't use `@import` in SCSS; use `@use` |
| Add i18n keys for all user-visible text | Don't hardcode Chinese/English strings |
| Auto-discover pages/sidebars/components by adding files | Don't manually edit barrel files for auto-discovered features |

## Touch map

| What | Path |
|------|------|
| Core API barrel | `financial-web/src/core/api/index.ts` |
| Core config | `financial-web/src/core/config/` |
| Core router | `financial-web/src/core/router/` |
| Core stores | `financial-web/src/core/stores/` |
| Modules | `financial-web/src/modules/` |
| Shell | `financial-web/src/shell/` |
| Domains | `financial-web/src/domains/` |
| Shared UI | `financial-web/src/shared/ui/` |
| Shared lib | `financial-web/src/shared/lib/` |
| Styles | `financial-web/src/styles/` |
| Locales | `financial-web/src/locales/` |
| Types (generated) | `financial-web/src/types/api/generated/schema.d.ts` |
| Bootstrap | `financial-web/src/app/bootstrap/` |
| Web core rule | `financial-web/docs/rules/core.md` |
| Modular structure rule | `financial-web/docs/rules/modular-structure.md` |
| Modules rule | `financial-web/docs/rules/modules.md` |
| Architecture rule | `financial-web/docs/rules/architecture.md` |
| API conventions rule | `financial-web/docs/rules/api-conventions.md` |
| i18n rule | `financial-web/docs/rules/i18n.md` |
| TypeScript rule | `financial-web/docs/rules/typescript.md` |

## Sequences

1. **New page:** Add `src/domains/<name>/pages/<Name>Page.vue` → add
   `src/core/router/modules/<name>/routes.ts` → add path constants to
   `src/shared/constants/routerPaths.ts` → add i18n keys → use `el-skeleton`
   for loading.
2. **New shared component:** Put in `src/shared/ui/{common,widgets}/`
   → `index.vue` + `index.ts` → auto-registered globally.
3. **New module:** Create `src/modules/<name>/` → wire in
   `src/app/bootstrap/` → add `docs/rules` if platform-level.

## Verify

```bash
# From financial-web/
pnpm check                   # type + lint + format + unit + architecture + contract + MSW coverage
pnpm deps:boundaries         # circular dependency + module boundary check
pnpm deps:rules-docs         # rules doc link audit
```

## SSOT links

| Doc | Path |
|-----|------|
| Core rule | `financial-web/docs/rules/core.md` |
| Modular structure | `financial-web/docs/rules/modular-structure.md` |
| Modules | `financial-web/docs/rules/modules.md` |
| Architecture | `financial-web/docs/rules/architecture.md` |
| API conventions | `financial-web/docs/rules/api-conventions.md` |
| i18n | `financial-web/docs/rules/i18n.md` |
| TypeScript | `financial-web/docs/rules/typescript.md` |
| Web AGENTS | `financial-web/AGENTS.md` |

## Related packs

- **00** — Workspace map (repo layout)
- **01** — Contracts SSOT (generated types, sync)
- **03** — API architecture (backend layering)
- **10** — Fullstack playbook (pair-update with web side)
- **11** — Verification (`pnpm check` details)
