---
name: financial-fullstack-expert
description: >-
  Financial full-stack expert knowledge pack for AI coding agents.
  Covers Vue 3 frontend (financial-web/) + FastAPI backend (financial-api/) +
  Admin awareness (financial-admin/). Load IDENTITY.md + ROUTER.md first,
  then 1-4 business packs per scenario. Use when editing any Financial code,
  contracts, navigation, realtime, RCC, crawler, worker, auth, or audit.
---

# Financial Fullstack Expert

Tool-agnostic expert knowledge for the Financial full stack.

## How to use

1. Read `IDENTITY.md` — identity, 20 red lines, `code: 0` mnemonic,
   three-process model, conflict priority.
2. Read `ROUTER.md` — find your scenario, load 1–4 business packs.
3. Read linked repo SSOT docs for deep details.

## Structure

```
IDENTITY.md       ← always load (identity + red lines)
ROUTER.md         ← always load (scenario → pack routing)
packs/00-11       ← business knowledge packs (load by scenario)
adapters/         ← tool-specific integration (generic / Cursor / Claude / Codex / ChatGPT)
CHANGELOG.md      ← pack-level change tracking
```

## Conflict priority

```text
sub-repo docs/rules/** (SSOT) > this expert pack > workspace rules > global skills
```
