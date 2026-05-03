# Craft Memory — Path, Layout, Invariants

Craft skills share one durable "memory" directory — the long-lived home for design specs and feature state. All three skills (`brainstorm`, `implement`, `feature-state`) read and write against this single location. This reference defines where it lives, what goes in it, and the invariants every skill must respect. Read this file once before any memory access in a session so all three skills stay consistent.

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
- The memory directory may not exist yet. Any skill that writes to memory must `mkdir -p` the target subdirectory (`specs/`, `features/<slug>/changes/`) before writing.

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
└── features/<slug>/                        # per-feature memory
    ├── overview.md                         # always loaded; canonical decisions + Recent Changes index
    ├── changes/                            # leaf files, on-demand load
    │   └── YYYY-MM-DD-<short-topic>.md     # 1-2 paragraphs; one per memorable implement session
    └── snapshots/                          # DEPRECATED; do not write to
        └── YYYY-MM-DD.md                   # legacy reference only; kept for archaeology
```

Throughout all craft SKILL.md files, `<memory-root>` refers to the path above. File-level roles, read protocol, and write protocol for `overview.md` and `changes/*.md` live in [`feature-files.md`](feature-files.md).

## Invariants

- **Specs are immutable.** Once written, never edit. If the design changes, write a new spec.
- **`overview.md` is mutable with consolidation discipline.** Both `craft:implement` and `craft:feature-state` may add, edit, merge, or remove entries when they spot duplicates or contradictions. Same auto-memory rule: do not write duplicates; update or remove outdated entries.
- **`changes/*.md` are immutable once written.** New facts go in a new dated file. Same-day collision: append `b`, `c`, etc.
- **`snapshots/*.md` are deprecated and read-only.** No new snapshots are written. Existing files remain on disk as legacy reference; do not edit, do not delete, do not migrate.
- **Code is the source of truth.** When `overview.md`, a changes file, or a legacy snapshot disagrees with current code, fix the memory (consolidate via `craft:feature-state` or write a new changes file) - do not "fix" the old file.
- **Load the anchor first.** A new session on an existing feature reads `overview.md` always; changes files load on demand only when the index entry or current task references one. Legacy `snapshots/` is never default-loaded.

## What NOT to put in memory

- Implementation plan files — craft uses TodoWrite for planning, not on-disk plans.
- Team-shareable architecture docs — those belong in the project's tracked `docs/` tree.
- Session scratch notes or in-progress drafts.
- Anything derivable from reading current code.
