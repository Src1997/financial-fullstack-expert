# Financial Fullstack Expert — Knowledge Pack

> **Tool-agnostic** expert knowledge for the Financial full stack
> (frontend `financial-web/` + backend `financial-api/` + admin `financial-admin/`).

## What this is

A modular knowledge pack that gives any AI coding agent the context it needs
to work safely across all three Financial repositories. It is **not** a
replacement for in-repo SSOT docs — it is a navigation, constraint, and
touch-map layer.

## Structure

```text
financial-fullstack-expert/
├── SKILL.md            ← auto-discovery entry (CatPaw / Cursor skill)
├── README.md           ← you are here
├── IDENTITY.md         ← always-loaded identity & red lines
├── ROUTER.md           ← scenario → pack routing table
├── packs/
│   ├── 00-workspace-map.md                ✅ P0
│   ├── 01-contracts-ssot.md               ✅ P0
│   ├── 02-web-architecture.md             ✅ P0
│   ├── 03-api-architecture.md             ✅ P0
│   ├── 10-fullstack-playbook.md           ✅ P0
│   ├── 11-verification.md                 ✅ P0
│   ├── 04-navigation-manifest.md          ✅ P1
│   ├── 05-realtime-i18n.md                ✅ P1
│   ├── 06-rcc-capability.md               ✅ P1
│   ├── 07-crawler-scripts-worker.md       ✅ P1
│   ├── 08-auth-audit.md                   ✅ P1
│   ├── 09-styles-tokens-theme.md          ✅ P1
│   └── 12-anti-patterns.md                ✅ P1 (index)
└── adapters/                              ✅ P2
    ├── generic-system-prompt.md
    ├── cursor.md
    ├── claude-code.md
    ├── codex.md
    └── chatgpt.md
```

## How to use

1. **Always load**: `IDENTITY.md` + `ROUTER.md`.
2. **Cold start**: add `packs/00-workspace-map.md`.
3. **Scenario match**: look up the routing table in `ROUTER.md`, load 1–4
   business packs.
4. **Read repo SSOT** for deep details — pack links tell you which file.
5. **Tool-specific setup**: pick an adapter from `adapters/` for your AI tool.

## Conflict priority

```text
sub-repo docs/rules/** (SSOT)
  > this expert pack
    > workspace rules (IDE-specific: .cursor/rules/, .catpaw/rules/, etc.)
      > personal global skills
```

## Delivery status (2026-10-08)

| Phase | Packs | Status |
|-------|-------|--------|
| P0 | IDENTITY + ROUTER + 00 + 01 + 02 + 03 + 10 + 11 | ✅ Complete |
| P1 | 04 + 05 + 06 + 07 + 08 + 09 + 12 | ✅ Complete (delivered 0.7.0) |
| P2 | Adapters (generic, Cursor, Claude Code, Codex, ChatGPT) | ✅ Complete |

**20 red lines** (was 11 in 0.2.0, 15 in 0.3.0, 18 in 0.5.0). All packs and
adapters delivered. P1 packs are distilled summaries — their SSOT links point
to the authoritative repo docs (`docs/rules/**`, `docs/skills/project/**`).

> **Methodology (v0.5.0+):** All red lines and findings now follow
> ruleskills-kit "Project First" — project's own `docs/rules/**` is the SSOT;
> external generic patterns are used only as supplementary inspiration.

### Key references added in 0.4.0-0.5.0

- `financial/docs/reviews/2026-08-18-admin-database-seed-supplement.md` —
  Admin control-plane tables (`fin_admin_role_capabilities`,
  `fin_config_revisions`, `fin_role_permission_bindings`,
  `fin_admin_sessions`), DB table design issues (TimestampMixin, plural
  names, JSONB dual-write), seed safety (weak passwords, purge, strategy consistency)
- `financial/docs/reviews/2026-08-18-unified-execution-plan.md` —
  **canonical Phase 0-6 execution sequence** (60 person-days, 11 weeks)

### Key references added in 0.3.0

- `financial/docs/reviews/2026-08-18-supplementary-refactoring-plan.md` —
  consolidated inventory of all non-compliant designs (shell→domains, shared
  God-object, startup migrate/seed, admin metadata leakage, audit gaps)
- `financial/docs/reviews/2026-08-18-seed-data-governance-redesign.md` —
  JSONB array → row-level table migration for seed/config data
