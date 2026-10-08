# Cursor Adapter — Financial Fullstack Expert

## How to use in Cursor

### Option A: Workspace rules (recommended)

Add a short rule in your workspace rules directory that forwards to the expert pack:

```text
# Financial Fullstack Expert
Before coding in financial-web/, financial-api/, or financial-admin/, read:
1. IDENTITY.md
2. ROUTER.md
Then load 1-4 business packs per the routing table.
Red lines in IDENTITY.md are non-negotiable.
```

> Adjust the root path based on where the pack is installed in your workspace.

### Option B: Auto-discovery

Tools that support `SKILL.md` frontmatter (e.g. CatPaw) auto-discover the
expert pack when the description matches the user's task context. No manual
wiring needed.

## What NOT to do

- Do not copy pack contents into workspace rules — they will go stale.
- Do not create a parallel rule system; the expert pack is the SSOT.
- Red lines from `IDENTITY.md` may be extracted as short rules if Cursor needs
  them for enforcement, but always link back to the full expert pack.

## Pack loading

Tools with `@file` mention support can load specific packs:

```text
@IDENTITY.md
@ROUTER.md
@packs/02-web-architecture.md
```

Keep the total loaded context within budget (≤ 4 business packs).

## Conflict priority

```text
sub-repo docs/rules/** (Normative SSOT)
  > this expert pack
    > workspace rules
      > personal global skills
```
