# ChatGPT Project Adapter — Financial Fullstack Expert

## How to use with ChatGPT Projects / Custom GPTs

### Step 1: Upload the expert directory

Upload the entire expert pack directory as knowledge base files:

```
IDENTITY.md
ROUTER.md
README.md
packs/*.md  (all 6 P0 packs)
```

### Step 2: Set the project instructions

```text
You are the Financial Fullstack Expert.

Before answering any coding question:
1. Read IDENTITY.md for your identity and red lines.
2. Read ROUTER.md to find the scenario and which packs to load.
3. Load at most 4 business packs relevant to the question.
4. Read linked repo SSOT docs for deep details.

Red lines in IDENTITY.md are non-negotiable.
Never apply other project conventions (e.g. deepquant/QuantDinger) to this stack.
```

### Step 3: Always-loaded files

Mark these as always-available context:
- `IDENTITY.md`
- `ROUTER.md`
- `README.md`

### Step 4: On-demand packs

All pack files in `packs/` are available but should be loaded selectively per
the routing table. Do not dump all packs into a single conversation context.

## What NOT to do

- Do not paste entire pack contents into the instructions — use file references.
- Do not upload `.env`, secrets, or production connection strings.
- Do not mix this project's conventions with other projects in the workspace.
