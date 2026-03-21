Audit all Claude commands and Cursor rules for drift and inconsistency, then sync them so both systems enforce the same standards.

## What "in sync" means

Each role has two representations:
- A **Cursor rule** in `.cursor/rules/<role>.mdc` — passive, always-on guidance injected into Cursor AI context
- A **Claude command** in `.claude/commands/<role>.md` — active, invokable skill that performs tasks

They are in sync when:
1. Every principle in the Cursor rule is reflected in the Claude command's behavior
2. Every constraint in the Claude command is present in the Cursor rule
3. Cross-skill rules are consistent — if `foundations` mandates `uv`, then `tester`, `backend-engineer`, and every other skill that touches Python or Dockerfiles also mandates `uv`
4. No role has a Claude command without a Cursor rule, or a Cursor rule without a Claude command

## Steps

### 1. Inventory
Read every file in:
- `.cursor/rules/*.mdc`
- `.claude/commands/*.md`

Build a pairing table:

| Role | Cursor rule | Claude command |
|---|---|---|
| foundations | ✓ / ✗ | ✓ / ✗ |
| tester | ✓ / ✗ | ✓ / ✗ |
| ... | | |

Flag any role that exists in one system but not the other — that is an **Orphan** and must be created.

### 2. Pairwise drift detection
For each paired role, compare the Cursor rule and Claude command side by side:
- Extract the key rules / constraints from each
- Identify anything present in the Cursor rule but absent from the Claude command
- Identify anything present in the Claude command but absent from the Cursor rule
- Report these as **Drift** items

### 3. Cross-skill consistency check
Read the `foundations` Cursor rule and Claude command to extract all project-wide mandates (e.g., "use `uv` not `pip`", "always containerize", "conventional commits").

Then scan every other Cursor rule and Claude command for:
- References that contradict a foundations mandate (e.g., `pip install` in a Dockerfile example)
- Gaps where a foundations mandate should be echoed but isn't (e.g., tester showing a `pip`-based Dockerfile)

Report these as **Cross-skill inconsistencies**.

### 4. Produce a Sync Report

```
## Skill Sync Report

### Orphans (missing pair)
- ...

### Drift — <role>
**In Cursor rule, missing from Claude command:**
- ...
**In Claude command, missing from Cursor rule:**
- ...

### Cross-skill inconsistencies
- <skill>: contradicts foundations rule "<rule>" — found "<offending text>"
- ...

### In sync
- ...
```

### 5. Apply fixes

For each issue found, propose the exact change (quoted diff-style: what to remove, what to add).

If `$ARGUMENTS` includes `--fix`, apply all changes automatically without prompting.
Otherwise, list proposed changes and ask the user to confirm before writing.

When creating a missing Cursor rule or Claude command, derive its content from the existing counterpart — do not invent new rules.

## Running on a schedule

This command should be run:
- When any Cursor rule or Claude command is modified
- Before merging changes that touch `.cursor/rules/` or `.claude/commands/`
- Any time a new role is added

## Arguments

- No arguments — audit and report only
- `--fix` — audit then apply all fixes automatically
- `--role <name>` — audit and fix a single role pair only
