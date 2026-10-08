# Pack 05 — Realtime + i18n + Data Sources

## When to load

- Adding a WebSocket channel, push feed, or live-updating UI.
- Wiring REST vs WebSocket refresh strategy for a page or panel.
- Adding locale keys, a new locale, or debugging bilingual/WS-lang issues.

## Scope / Out of scope

**Scope:** REST vs WS refresh strategy, WS topology (single client +
multiplexed channels), i18n three-context model, copy ownership, locale
propagation, lang-scoped channels.

**Out of scope:** Contract file mechanics (→ pack 01), home block loaders
(→ pack 02 + `data-sources.md`), backend crawler push internals (→ pack 07).

## Must know

### Data refresh strategy (REST vs WebSocket)

Do **not** poll REST endpoints with `setInterval` for live UI.

| Strategy | When | Pattern |
| -------- | ---- | ------- |
| REST on mount | Page/sidebar loads | `useAsyncData` / store `load()` in `onMounted` or panel open |
| REST manual refresh | User action | Expose `refresh()` from composable or store |
| WebSocket | High-frequency / push (quotes, bars, chat, flash news) | `@/modules/websocket` subscribe helpers; subscribe on mount, unsubscribe on unmount |

- Fetch only when the owning module is mounted; shell chrome must not poll
  domain REST APIs (`App.vue` must not call market APIs — the shell ticker owns
  its quote sources).
- Home blocks: granular REST on viewport activation; flash feed subscribes to
  `headlines` WS for incremental items.
- Hidden panels use `v-if` (not `v-show`) so subscriptions actually stop.

### WS topology

One shared `WsClient` to `/api/ws` with multiplexed channels
(`quotes`, `bars`, `chat`, `qa`, `headlines`, `calendar`, `signals`);
reference-counted subscriptions on that single socket.

Add a **second connection** only for isolation (different auth boundary,
third-party endpoint, backpressure). Otherwise add a **channel/topic** on the
existing client — which means updating
`packages/contracts/realtime-topics.json` → `pnpm sync:realtime-topics` →
backend `topic_registry.py` (generated block) → facade in `modules/websocket`.

### i18n three contexts

| Context | API | Rule |
| ------- | --- | ---- |
| `<template>` | `$t('key')` | All inline UI text |
| `<script setup>` | `const { t } = useI18n()` | Only when script logic needs it |
| Pure `.ts` | `import { t } from '@/modules/i18n'` | Call-site runtime only, never top-level |

Forbidden: importing `t` from `@/modules/i18n` inside `.vue`; top-level
frozen translated strings; local ternary translation maps
(`isZh ? 'Chinese' : 'English'`).

### Copy ownership

- Frontend i18n owns: system chrome, route titles, notifications, validation,
  shell labels, aria labels, placeholders.
- Backend owns via `Accept-Language`: article/calendar titles and bodies,
  category/symbol display names, user-generated content. Frontend only passes
  the locale.

### Locale propagation (exactly two channels)

1. `Accept-Language` header on every request (default).
2. `?lang=` query only for cache-sensitive public GET endpoints (`localeQuery()`).

Never add a third custom header/body field. Locale is mutated only via
`setAppLocale` (single source of truth — no second writable ref).

### WS + locale alignment

- Shared `idMatchesLocale` predicate for bilingual id filtering.
- Clear/filter WS incremental buffers on locale switch.
- Refetch on locale switch via `localeSensitive: true` (or
  `watch(appLocale, refresh)`).
- Lang-scoped channels (`headlines`, `calendar`, `signals`) pass `lang` on
  subscribe; payloads are guarded client-side.

## Do / Don't

| Do | Don't |
|----|-------|
| Add push feeds as WS channels + sync | Don't add REST polling instead |
| Store raw i18n keys in configs; translate at render | Don't hardcode Chinese/English copy |
| Use `idMatchesLocale` for bilingual filtering | Don't hand-roll lang checks per component |
| New keys in both `zh-CN.ts` + `en-US.ts` | Don't ship single-locale keys |

## Verify

```bash
# From financial-web/
pnpm check                                  # includes i18n / MSW coverage audits
pnpm sync:realtime-topics                   # after editing realtime-topics.json
pnpm deps:realtime-topics                   # web ↔ api topic parity audit
```

Manual: switch locale → verify lists refetch, WS buffers clear, no reload hack.

## Touch map

| What | Path |
| ---- | ---- |
| Realtime topics SSOT | `financial-web/packages/contracts/realtime-topics.json` |
| Topic registry (web) | `financial-web/src/modules/websocket/topic-registry.ts` |
| Topic registry (api) | `financial-api/app/realtime/topic_registry.py` |
| i18n facade | `financial-web/src/modules/i18n/` |
| Locale modules | `financial-web/src/locales/modules/<group>/<name>/` |
| Data-source rule | `financial-web/docs/rules/data-sources.md` |
| i18n rule | `financial-web/docs/rules/i18n.md` |
| Backend realtime rule | `financial-api/docs/rules/modules/realtime/realtime.md` |
| Bilingual data modes | `financial-api/docs/rules/core/i18n-data-modes.md` |

## SSOT links

| Doc | Path |
| --- | ---- |
| Data sources | `financial-web/docs/rules/data-sources.md` |
| i18n | `financial-web/docs/rules/i18n.md` |
| Realtime skill | `financial-web/docs/skills/project/financial-web-realtime/` |
| i18n skill | `financial-web/docs/skills/project/financial-web-i18n/` |
| i18n data modes (api) | `financial-api/docs/rules/core/i18n-data-modes.md` |

## Related packs

- **01** — Contracts SSOT (realtime-topics sync chain)
- **02** — Web architecture (module boundaries, home loaders)
- **07** — Crawler + Worker (who produces the pushes)
