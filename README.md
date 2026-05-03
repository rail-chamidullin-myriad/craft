# Craft

Skills for developing features continuously across sessions, instead of treating each task as a one-off build.

Two main skills carry feature work end to end:

- **`craft:brainstorm`** turns an idea into a fully formed design through a one-question-at-a-time dialogue, then writes a dated spec on disk.
- **`craft:implement`** picks up that spec (or the current chat, when the work is small) and ships it with a worktree, a TodoWrite ledger, and TDD cadence.

What makes them work *continuously* is **feature memory**: a small always-loaded `overview.md` per feature plus per-session leaf files, modeled on Claude Code's auto-memory pattern. Anything non-trivial spans many sessions, and by the fifth one a Claude Code session normally spends real context re-deriving what was already decided. Feature memory means each new brainstorm or implement session starts from one short file instead of the full spec pile - and `craft:implement` keeps that file fresh by writing a short note to it at finish.

A third skill, `craft:feature-state`, compacts the memory on demand when it drifts.

## Skills

- **`craft:brainstorm`** - Spec-driven design dialogue. Loads prior feature memory, asks clarifying questions one at a time (with a recommended answer where useful), proposes 2-3 approaches with trade-offs, and presents the design in sections for incremental approval. Optional browser Visual Companion for UI mockups, diagrams, and visual A/B selection. Output: a dated spec.
- **`craft:implement`** - Execute a feature in the current session. Two modes: *from-spec* (read a design spec from disk) or *from-conversation* (skip the spec when you already know what to build). Sets up a `.worktrees/<branch>/` worktree, bootstraps `features/<slug>/` memory if missing, follows a TDD cadence, and at finish writes a `changes/*.md` leaf and proposes any canonical-decision diffs to `overview.md`.
- **`craft:feature-state`** - Compact a feature's `overview.md` when it accumulates duplicates, contradictions, or stale entries. On-demand, auto-dream-shaped.

## Feature memory

The differentiator. Memory lives outside the working tree at:

```
~/.claude/projects/<project-slug>/craft/memory/
```

`<project-slug>` is the project's absolute path with `/` replaced by `-` (matching Claude Code's native convention). The path is derived at runtime; no configuration, no override.

```
<memory-root>/
├── specs/                                  # dated design specs from brainstorming sessions
└── features/<slug>/
    ├── overview.md                         # always loaded: canonical decisions + Recent Changes index (FIFO@10)
    └── changes/                            # leaf files, loaded on demand only
        └── YYYY-MM-DD-<short-topic>.md     # 1-2 paragraphs per memorable implement session
```

`overview.md` is the always-loaded anchor (hard cap 5k chars). It holds long-lived canonical decisions (product/business rules, architectural invariants, external constraints) plus a FIFO index of the 10 most recent changes files. Each `changes/*.md` leaf is 1-2 paragraphs, capped at 2k chars, and only loaded when relevant.

Discipline comes from scope: a changes file describes one session's delta, by construction it cannot grow into a full state dump. Compaction is opportunistic - both `craft:implement` (at finish) and `craft:feature-state` (on demand) clean up duplicates and contradictions as they go.

**Why outside the working tree:**
1. Git worktrees share memory automatically, since it's not inside any tree.
2. Personal brainstorming artifacts never pollute `git status` or PRs.
3. It sits as a sibling to Claude Code's built-in auto-memory (`~/.claude/projects/<slug>/memory/`) - same category, same parent.

Full spec: [`references/memory.md`](references/memory.md).

Two other directories are created at the repo root and should be gitignored:

- `.worktrees/` - isolated worktrees created by `craft:implement`.
- `.craft/brainstorm/` - persistent Visual Companion session data.

## Installation

Install from this repo via the Claude Code plugin marketplace. In a Claude Code session:

```
/plugin marketplace add rail-chamidullin-myriad/craft
/plugin install craft@craft
```

The marketplace is named `craft` and publishes one plugin also named `craft` - hence `craft@craft`.

Manage:

```
/plugin list
/plugin marketplace update craft     # pull newer versions
/plugin uninstall craft@craft        # remove
```

### Local development install

If you're hacking on craft itself, point the marketplace at your local checkout:

```
/plugin marketplace add ./
/plugin install craft@craft
```

Re-run `/plugin marketplace update craft` after edits to pick up changes.

## Origin

Craft started as a fork of [Superpowers](https://github.com/obra/superpowers) and went its own direction once feature memory became the central concept. The brainstorming flow (one-question-at-a-time, 2-3 approaches with trade-offs, sections-with-approval, dated spec) and the spec template are inherited from Superpowers; the Visual Companion scripts under `skills/brainstorm/scripts/` are vendored directly (see `CREDITS.md`). Implementation diverges - craft drops the separate plan-on-disk artifact in favor of a TodoWrite ledger in the main session loop, and adds the per-feature memory layer that Superpowers does not have.

The "surgical changes only" rule in `craft:implement` is informed by [Andrej Karpathy's observations on LLM coding](https://x.com/karpathy/status/2015883857489522876).

### Upstream sync (for maintainers)

Visual Companion scripts are vendored from `superpowers:brainstorming`. To check for upstream improvements:

```bash
diff -r ~/.claude/plugins/cache/claude-plugins-official/superpowers/<latest>/skills/brainstorming/ ./skills/brainstorm/
```

Read the diff, port selectively.

## License

Personal workflow skills. Use, adapt, ignore - whatever fits your flow.
