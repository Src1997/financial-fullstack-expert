# Codex / AGENTS.md Adapter — Financial Fullstack Expert

## How to use with Codex or any AGENTS.md-aware tool

Create or update `AGENTS.md` at the repo root or workspace root.

### Recommended `AGENTS.md` content

```markdown
# Agents — Financial

## Expert knowledge pack
This project uses a modular expert pack.

### Before coding
1. Read IDENTITY.md
2. Read ROUTER.md
3. Load 1-4 business packs per the routing table in ROUTER.md

### Red lines
See IDENTITY.md "Red lines" — non-negotiable for all agents.

### Conflict priority
sub-repo docs/rules (SSOT) > this expert pack > workspace rules
```

> Adjust the root path based on where the pack is installed in your workspace.

## Multi-repo note

If the tool requires per-repo `AGENTS.md`, place a forwarding copy in each:
- `financial-web/AGENTS.md` → points to the expert pack
- `financial-api/AGENTS.md` → points to the expert pack
- `financial-admin/AGENTS.md` → points to the expert pack

Keep each `AGENTS.md` under 30 lines. All detail lives in the packs.

## What NOT to do

- Do not copy pack contents into `AGENTS.md`.
- Do not duplicate SSOT docs into the expert packs or adapter files.
