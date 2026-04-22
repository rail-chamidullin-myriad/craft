# Craft Memory — Path, Layout, Invariants

Craft skills share one durable "memory" directory — the long-lived home for design specs, task logs, and feature state. All three skills (`brainstorm`, `implement`, `feature-state`) read and write against this single location. This reference defines where it lives, what goes in it, and the invariants every skill must respect. Read this file once before any memory access in a session so all three skills stay consistent.

## Path

Craft memory always lives at:

```
$HOME/.claude/projects/<project-slug>/craft/memory/
```

- **`<project-slug>`** is the current project's absolute path with `/` replaced by `-`. For `/Users/alice/Projects/myriad` the slug is `-Users-alice-Projects-myriad`. Same encoding Claude Code uses for `~/.claude/projects/<slug>/*.jsonl` session logs and auto-memory.
- **Derive the slug at runtime. Do not hardcode it.** Portable snippet:
  ```bash
  PROJECT_ROOT="$(git rev-parse --show-toplevel 2>/dev/null || pwd)"
  PROJECT_SLUG="${PROJECT_ROOT//\//-}"
  MEMORY_ROOT="$HOME/.claude/projects/${PROJECT_SLUG}/craft/memory"
  ```
- The memory directory may not exist yet. Any skill that writes to memory must `mkdir -p` the target subdirectory (`specs/`, `tasks/`, `features/<slug>/`) before writing.

## Why outside the working tree

Memory deliberately lives outside the git working tree. Three reasons:

1. **Worktree sharing.** Git worktrees each have their own working tree; files inside the tree don't sync across them. Memory outside the tree is one canonical location every worktree reads and writes.
2. **No git pollution.** Specs and brainstorming artifacts are personal one-question-at-a-time dialogues, not team-shareable material. Keeping them out of the tree keeps `git status` and PR diffs clean.
3. **Sibling to auto-memory.** `~/.claude/projects/<slug>/memory/` is Claude Code's built-in per-project memory. `~/.claude/projects/<slug>/craft/memory/` is craft's equivalent. Same conceptual category, same parent tree.

## Layout

```
<memory-root>/
├── specs/                                  # dated design specs from craft:brainstorm
│   └── YYYY-MM-DD-<topic>-design.md        # immutable; never edit in place
├── tasks/                                  # dated session logs from craft:implement
│   └── YYYY-MM-DD-<topic>.md               # one per implementation session
└── features/<slug>/                        # per-feature distilled state
    ├── overview.md                         # stable, append-only canonical decisions
    └── YYYY-MM-DD-<slug>-state.md          # dated snapshots; newest is authoritative
```

Throughout all craft SKILL.md files, `<memory-root>` refers to the path above.

## Invariants

- **Specs are immutable.** Once written, never edit. If the design changes, write a new spec (and optionally a new feature-state snapshot that supersedes the old one).
- **State snapshots accumulate.** Dated filenames, newest wins, keep all older ones as an audit trail. Never delete.
- **`overview.md` is append-only.** Stable long-lived canonical decisions. Append only on explicit user confirmation. Never mass-rewrite.
- **Code is the source of truth.** When a spec or state file disagrees with current code, re-distill via `craft:feature-state` — do not edit the old spec or state file to "fix" it.
- **Load the newest first.** A new session on an existing feature reads `features/<slug>/overview.md` plus the newest `YYYY-MM-DD-<slug>-state.md` only. Older state files and older specs are for archaeology, not default context.

## What NOT to put in memory

- Implementation plan files — craft uses TodoWrite for planning, not on-disk plans.
- Team-shareable architecture docs — those belong in the project's tracked `docs/` tree.
- Session scratch notes or in-progress drafts.
- Anything derivable from reading current code.
