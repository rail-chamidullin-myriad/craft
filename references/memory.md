# Craft memory

Craft memory holds design specs and one `overview.md` per feature. It lives outside the working tree, so every worktree of the project shares it and it never shows up in `git status`.

## Path

```bash
PROJECT_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
MEMORY_ROOT="$HOME/.claude/projects/${PROJECT_ROOT//\//-}/craft/memory"
```

Derive it at runtime, next to Claude Code's own `~/.claude/projects/<slug>/memory/`. Create a folder with `mkdir -p` before the first write.

## Layout

```
<memory-root>/
├── specs/YYYY-MM-DD-<topic>-design.md      # one per brainstorm, never edited after
└── features/<feature>/overview.md          # a leaf, or a hub when it holds sub-features
    └── <sub-feature>/overview.md           # at most two levels
```

Earlier craft versions also wrote `changes/` and `snapshots/` folders. Craft no longer reads or writes them. Leave them on disk unless the user asks to delete them.

## Reading

- Read the feature's `overview.md` in full at the start of a session.
- Open a spec only when `overview.md` or the user points at it.
- Follow a Related Features link one level deep, and only when the task needs that feature.

## overview.md

It loads every session, so it holds only what outlives the code. Keep it under 5k characters.

| Section | Holds |
|---|---|
| Product / Business | Why the feature exists, who it serves, scope set by stakeholders |
| Architectural Invariants | Rules future code must keep |
| External Constraints | Compliance, SLAs, contracts with other teams |
| Related Features | `- <slug> - <the seam> [owns: <area>]`, a leaf linking up to its hub |
| Recent Changes | `- YYYY-MM-DD: <topic> -> specs/<file>.md`, newest first, last 10 |

A hub holds a one-line blurb and Related Features only.

Rules for each line:

- **Constraint, not description.** Ask: would the line need rewriting if the implementation changed tomorrow? If yes, drop it or reword it to the rule underneath. "Retention is set per plan, never per report" is a constraint. "Retention lives in the `plans` table" is a description.
- **One fact per line**, under 200 characters, ending with `Source: specs/<file>.md`.
- **The user confirms** every constraint line. Recent Changes is mechanical.
- **No duplicates.** A new decision replaces the line it contradicts.
- **The owner states it once.** A cross-cutting rule lives in the feature marked `owns:`, and the others link to it.

Template:

```markdown
# <Feature> - Overview

## Product / Business
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md

## Architectural Invariants
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md

## External Constraints
- <Constraint>. Source: <spec or external doc>

## Related Features
- <hub> - domain index.

## Recent Changes
- YYYY-MM-DD: <topic> -> specs/YYYY-MM-DD-<topic>-design.md
```

## New features

Place a new feature under the hub its code and topic belong to, or at the top level. Ask once when the placement is ambiguous, with your inferred answer.

## Compaction

When `overview.md` passes 4k characters, or a session spots duplicates or contradictions:

1. Drop descriptions and anything the code answers.
2. Merge duplicates and resolve contradictions against the current code.
3. Fold any other section into the five above, or drop it.
4. Shorten the survivors.

Show changes in meaning to the user as a short diff and apply them on a yes. Rewording needs no confirmation.
