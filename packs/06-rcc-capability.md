# Pack 06 — RCC + Capability Activation

## When to load

- Making any user-facing value admin-tunable (brand, layout, flags, ads).
- Deciding whether a config is Feature-L or Platform-L.
- Adding a new runtime config key end-to-end (api → admin → web).

## Scope / Out of scope

**Scope:** RCC snapshot pipeline, read priority, key namespaces, capability
model (Feature-L vs Platform-L), maturity levels L0–L4, new-key checklist.

**Out of scope:** Admin form UX (→ admin rules), contract sync mechanics
(→ pack 01), auth/RBAC internals (→ pack 08).

## Must know

### Pipeline

```text
fin_app_configs (DB) -> GET /system/runtime-config -> src/modules/runtime-config -> getRuntimeConfig()
```

Read priority:

```text
fresh API payload > localStorage snapshot (fb:runtime-config) > caller fallback
```

- Snapshot hydrates synchronously at module init (footer, navigation, first paint).
- `refreshRuntimeConfig` sends `If-None-Match` with current `revision`;
  `304` / empty body is a no-op and must not wipe in-memory state.
- API failures are silent; consumers keep snapshot or fallback values.

### Capability-activation model (root normative)

**If a configuration can be adjusted via Admin, it must never require a
deployment to change.** Code defines what capabilities exist; Admin controls
which are active and how they behave at runtime.

| Layer | Controls | Examples | Managed via |
| ----- | -------- | -------- | ----------- |
| **Feature-L** | User-facing activation/layout | home blocks, nav toggles, ad placements, brand text | `config.write` (content ops forms) |
| **Platform-L** | Security/access control | role-capability matrix, feature flags, menu permission map | `policy.write` (platform admins) |

This separation prevents content operators from changing access control.

### Maturity levels

| Level | Description | Example |
| ----- | ----------- | ------- |
| L0 | Hardcoded; requires deployment | theme list in source |
| L1 | RCC raw JSON, no form | `ops.backlog_threshold` |
| L2 | RCC structured (Pydantic schema), no form | `market.trend_column_kinds` |
| L3 | Admin dedicated form + RCC storage | brand settings, home layout |
| L4 | Form + version history + diff + audit | target for critical configs |

New features launch at **L1 minimum**; L0 only for truly static concerns
(e.g. SCSS token names).

### Key namespaces (examples)

`brand.name`, `brand.logo_url`, `links.footer_columns`, `links.deepquant`,
`navigation.feature_flags`, `home.ads_enabled`, `ads.master_enabled`,
`shell.global_ads_enabled`, `shell.global_ad_placements`. Typed shapes live in
`RuntimeConfigPayload` (`src/modules/runtime-config/types`).

### Adding a new configurable key (5 steps)

1. **API**: register key in `CONFIG_DEFINITIONS`
   (`app/domains/system/app_config/registry.py`) with Pydantic schema.
2. **API**: add seed fixture in `app/db/fixtures/config/app.py`.
3. **Admin**: dedicated form (L3) or raw JSON editor (L1/L2) via
   `PUT /api/system/app-config`.
4. **Web**: consume via `getRuntimeConfig('namespace.key', fallback)` in
   `@/modules/runtime-config` — never call the HTTP endpoint from pages.
5. Verify: `uv run python -m app.db.seed` + `pnpm dev`.

## Do / Don't

| Do | Don't |
|----|-------|
| Read via `getRuntimeConfig(key, fallback)` with a code fallback | Don't call the runtime-config HTTP endpoint from pages/components |
| Classify Feature-L vs Platform-L before adding | Don't let content ops change policy via `config.write` |
| Launch new tunables at L1+ | Don't hardcode values that already have runtime keys |
| Keep `admin.*` / `navigation.*` / role-matrix keys `is_public=False` | Don't leak platform metadata into the anonymous snapshot |

## Touch map

| What | Path |
| ---- | ---- |
| RCC registry | `financial-api/app/domains/system/app_config/registry.py` |
| Admin CRUD | `financial-api/app/domains/system/app_config/router.py` |
| Config fixtures | `financial-api/app/db/fixtures/config/app.py` |
| Web runtime-config | `financial-web/src/modules/runtime-config/` |
| Bootstrap wiring | `financial-web/src/app/bootstrap/providers/setupRuntimeConfig.ts` |
| Web rule | `financial-web/docs/rules/runtime-config.md` |
| API rule | `financial-api/docs/rules/modules/app_config.md` |
| Capability model | `financial/docs/rules/capability-activation.md` |

## SSOT links

| Doc | Path |
| --- | ---- |
| Capability-activation (normative) | `financial/docs/rules/capability-activation.md` |
| Web RCC rule | `financial-web/docs/rules/runtime-config.md` |
| API app_config rule | `financial-api/docs/rules/modules/app_config.md` |
| Runtime-config skill | `financial-web/docs/skills/project/financial-web-runtime-config/` |
| Admin conventions | `financial-admin/docs/rules/conventions.md` |

## Related packs

- **01** — Contracts SSOT (admin-capabilities sync chain)
- **08** — Auth + Audit (Platform-L enforcement, `policy.write`)
