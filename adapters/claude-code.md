# Claude Code Adapter — Financial Fullstack Expert

## How to use with Claude Code

Create or update `CLAUDE.md` at the workspace root or at the repo root
(e.g. `financial/CLAUDE.md`).

### Recommended `CLAUDE.md` content

```markdown
# Claude Code — Financial

## Expert knowledge pack
Before coding in financial-web/, financial-api/, or financial-admin/:
1. Read IDENTITY.md
2. Read ROUTER.md
3. Load 1-4 business packs per the routing table

## Red lines
See IDENTITY.md "Red lines" section.
These are non-negotiable.

## Conflict priority
sub-repo docs/rules > this expert pack > workspace rules > global skills
```

> Adjust the root path based on where the pack is installed in your workspace.

## What NOT to do

- Do not copy pack contents into `CLAUDE.md` — use pointers only.
- Do not create a parallel convention system.
- `CLAUDE.md` should stay under 30 lines; all detail lives in the packs.

## Loading packs

Claude Code reads files on demand. Reference packs by relative path:

```
Read packs/03-api-architecture.md
```

Keep the session within budget (≤ 4 business packs excluding always-loaded).
