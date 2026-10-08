# Pack 09 — Styles + Tokens + Theme

## When to load

- Writing or reviewing any component SCSS / CSS variables.
- Adding colors, spacing, sizes, breakpoints, or a new theme.
- Debugging trend-color (green-up / red-up) or theme-switch behavior.

## Scope / Out of scope

**Scope:** Rule zero + token selection order, layered token model, forbidden
semantic aliases, theme ownership (`data-theme` / `data-trend`), component
style template, responsive policy.

**Out of scope:** Component layering / imports (→ pack 02), theme-pack
release SOP (`financial/docs/guides/theme-pack-release.md`), verification
(→ pack 11).

## Must know

### Rule zero

Do not hardcode visual constants in pages/components. Use:

- `var(--color-*)` for text/background/borders/semantic states.
- `var(--color-market-up)` / `var(--color-market-down)` for trend colors —
  they adapt automatically to the user's green-up / red-up scheme.
- `var(--fb-*)` (FastBull shell vars), `var(--spacing-*)`, `var(--font-size-*)`,
  `var(--radius-*)`, `var(--shadow-*)`, `var(--opacity-*)`, `var(--pct-*)`,
  `var(--size-*)`, `var(--viewport-*)`, `var(--z-index-*)`.
- Canonical layout names (`--chat-panel-height`) — **no** `--page-*` /
  `--panel-*` / `--dialog-*` aliases.
- `var(--flex-min)` instead of bare `min-width: 0` in flex children.
- SCSS variables/mixins from `src/styles/abstracts`.

Raw hex, repeated px spacing, ad-hoc breakpoints, and double fallback
`var(--a, var(--b))` in scoped styles are forbidden.

### Token selection order

1. **Existing token** — reuse `var(--spacing-4)` etc.
2. **Generic scales** — edit `tokens/primitives/` (opacity/pct/radius also
   register in `tokens/maps/_primitive-maps.scss`).
3. **Layout/component sizes** — add in `tokens/semantics/` → register in
   `tokens/maps/_semantic-maps.scss`.
4. **Theme colors** — edit `[data-theme]` blocks in `themes/_*.scss` only.
5. **Theme-only computed values** — `clamp()`/`min()`/`var()` combos in
   `_fastbull.scss`; never invent a second variable name.

### Layer model (one value → one variable name)

| Layer | Source | `:root` export | Varies by theme |
| ----- | ------ | -------------- | --------------- |
| Layout/Component | `tokens/semantics/` → `_semantic-maps.scss` | full `--*` name | no |
| Primitive scales | `tokens/primitives/` + `tokens/maps/_primitive-maps.scss` | `emit-all-root-tokens` | no |
| Theme/Color | `themes/_*.scss` | `[data-theme]` | yes |

Forbidden semantic aliases: chained re-exports (`--home-layout-gap:
var(--spacing-2)`), duplicate aliases (`--skeleton-w-60` vs `--pct-60`),
second domain prefixes, double fallback in components. `:root` may only hold
min/clamp/vw combinations that cannot be precomputed.

### Theme runtime contract

- Apply via `<html data-theme="...">`; default `fastbull-dark`; without a
  stored preference JS must **not** fall back to light.
- Persist under `financial-web:theme-mode`; stored value wins.
- Shell UI uses `useTheme` / `useTrendColor` from `@/shell/composables` —
  never touch `document.documentElement` or localStorage from components.
- Trend mode: `data-trend="green-up" | "red-up"`; components consume only
  `var(--color-market-up)` / `var(--color-market-down)`.
- Element Plus mapping stays in `src/styles/vendors/_element.scss`.
- **No Tailwind.** Responsive columns follow `responsive.md`
  (shell vs viewport vs main-content).

### Component style template

```vue
<style lang="scss" scoped>
@use '@/styles/abstracts' as *;

.feature-panel {
  display: flex;
  flex-direction: column;
  gap: var(--spacing-4);
  min-width: var(--flex-min);
  opacity: var(--opacity-medium);

  &__scroll {
    flex: 1 1 0;
    min-height: var(--flex-min);
    overflow-y: auto;
  }
}
</style>
```

Interactive transitions use motion mixins — no bare `transition: all`.
Breakpoint `@media` requires `@use '@/styles/abstracts' as *;` at top.

## Do / Don't

| Do | Don't |
|----|-------|
| Pick tokens in the 5-step order | Don't hardcode hex/px/z-index in scoped styles |
| Colors/shadows in `themes/_*.scss` only | Don't branch on theme names in components |
| Switch theme/trend via `useTheme`/`useTrendColor` | Don't write DOM attributes or localStorage directly |
| Register new tokens in maps + emit | Don't create ad-hoc `:root` blocks or alias chains |

## Verify

```bash
# From financial-web/
pnpm deps:style          # style/token audit
pnpm check               # full gate (includes design-token audit specs)
```

Manual: switch theme + trend mode → verify market colors and surfaces adapt.

## Touch map

| What | Path |
| ---- | ---- |
| Token sources | `financial-web/src/styles/abstracts/tokens/{primitives,mechanics,semantics}/` |
| Emit maps | `financial-web/src/styles/abstracts/tokens/maps/` |
| Theme files | `financial-web/src/styles/themes/_*.scss` (+ `_trend-colors.scss`) |
| Theme facade | `financial-web/src/modules/theme/` |
| Shell composables | `financial-web/src/shell/composables/useTheme.ts`, `useTrendColor.ts` |
| Element Plus mapping | `financial-web/src/styles/vendors/_element.scss` |
| Rules | `financial-web/docs/rules/styles.md`, `tokens.md`, `theme.md`, `responsive.md` |

## SSOT links

| Doc | Path |
| --- | ---- |
| Styles | `financial-web/docs/rules/styles.md` |
| Tokens | `financial-web/docs/rules/tokens.md` |
| Theme | `financial-web/docs/rules/theme.md` |
| Responsive | `financial-web/docs/rules/responsive.md` |
| Styles skill | `financial-web/docs/skills/project/financial-web-styles/` |
| Theme skill | `financial-web/docs/skills/project/financial-web-theme/` |

## Related packs

- **02** — Web architecture (where styles live in the layering)
- **06** — RCC (brand/logo keys consumed by shell chrome)
