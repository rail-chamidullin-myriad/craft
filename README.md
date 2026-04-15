# Craft

Disciplined, spec-driven software development skills for Claude Code.

Craft started as a few small edits to Superpowers and turned into its own thing.

What it keeps: the Superpowers brainstorming flow, which is great - one question at a time, 2-3 approaches with trade-offs, design presented in sections for incremental approval, and a dated spec on disk at the end.

What it improves:

First, the Superpowers implementation flow has a lot of steps that take time and consume tokens: design spec → implementation plan → execution. Having an implementation plan on top of a design spec doesn't necessarily produce better results - the plan mostly restates the spec, and the subagent-per-task loop exists to hand work to fresh subagents when most implementation fits better in the main session, where context and course-correction are cheap. Craft drops the plan: TodoWrite carries the task ledger, subagents come in only for isolated mechanical tasks, and the agent is instructed to pause and ask follow-up questions whenever something isn't clear rather than guessing.

Second, anything non-trivial spans multiple sessions, and by spec five each session spends real context re-deriving what was already decided. Craft adds per-feature artifacts: a dated state snapshot and an `overview.md` of canonical decisions. The next session starts from one file instead of the whole spec pile.

Everything else - the spec template, the TDD cadence, the Visual Companion - is inspired by Superpowers. The Visual Companion scripts under `skills/brainstorm/scripts/` are vendored directly; see `CREDITS.md`.

## Skills

- **`craft:brainstorm`** - Collaborative, spec-driven brainstorming. Loads prior feature state, asks clarifying questions one at a time (with a recommended answer where useful), proposes 2-3 approaches with trade-offs, and presents the design in sections for incremental approval. Optional browser Visual Companion for UI mockups, diagrams, and visual A/B selection. Produces a dated spec.
- **`craft:implement`** - Execute a feature in the current session. Two modes: *from-spec* (read a design spec from disk and implement it) or *from-conversation* (implement directly from the current chat, no spec file - for when you already know exactly what to do). Creates a `.worktrees/<branch>/` worktree for isolation (verifies `.worktrees/` is gitignored, runs project setup, confirms a clean test baseline), uses TodoWrite as the task ledger, and follows a TDD cadence (failing test → run → impl → run → commit). Writes a dated task log at finish. Optionally prompts to update feature state and overview.
- **`craft:feature-state`** - Distill a feature's current state into a dated snapshot so future brainstorming sessions can load one file instead of every historical spec.

## Directory layout

Artifacts default to `docs/craft/` at the repo root:

```
docs/craft/
├── specs/           # dated design specs from brainstorming sessions
├── tasks/           # dated implementation-session logs
└── features/<slug>/
    ├── overview.md              # stable, append-only canonical decisions
    ├── YYYY-MM-DD-<slug>-state.md      # newest snapshot is authoritative
    └── YYYY-MM-DD-<slug>-state.md      # older, kept for history
```

Override the artifact root in your project's `CLAUDE.md` (or equivalent project instructions file) if you want artifacts somewhere else (e.g. a personal subdirectory).

Two other directories are created at the repo root and should be gitignored:

- `.worktrees/` - isolated worktrees created by `craft:implement`.
- `.craft/brainstorm/` - persistent Visual Companion session data (mockups, state).

## Installation

Install from this repo via the Claude Code plugin marketplace. In a Claude Code session:

```
/plugin marketplace add rail-chamidullin-myriad/craft
/plugin install craft@craft
```

The marketplace is named `craft` (see `.claude-plugin/marketplace.json`) and publishes one plugin also named `craft` - hence `craft@craft`.

Verify and manage:

```
/plugin list
/plugin marketplace update craft     # pull newer versions
/plugin uninstall craft@craft        # remove
```

### Local development install

If you're hacking on craft itself, point the marketplace at your local checkout instead. A relative path works fine (resolved from the session's current working directory):

```
/plugin marketplace add ./
/plugin install craft@craft
```

Use an absolute path if you're running Claude Code from somewhere other than the parent directory. Re-run `/plugin marketplace update craft` after edits to pick up changes.

## Why not just use superpowers?

If you already know Superpowers, here's how craft differs:

| Superpowers | Craft |
|-------------|-------|
| `brainstorming` → spec on disk | `craft:brainstorm` → spec on disk (same shape; adds feature-state preload and spec self-check; hands off to `craft:implement`) |
| `writing-plans` → plan file on disk | (skipped entirely) |
| `executing-plans` / `subagent-driven-development` → plan re-read in new session, subagent per task | `craft:implement` (from-spec) → read spec directly, main session loop, TodoWrite ledger |
| No shortcut for "I know exactly what to do, just implement it" | `craft:implement` (from-conversation) → restate agreed scope inline, confirm, execute; no spec file needed |
| No feature-state concept | `craft:feature-state` + `features/<slug>/overview.md` to bound context across large multi-spec features |
| `finishing-a-development-branch` | (skipped - `craft:implement` ends with verify + task log, then a neutral worktree hand-off menu) |

Underlying disciplines (one-question-at-a-time, propose 2-3 approaches, present design in sections, TDD cadence, file-structure pre-flight, no-placeholder, spec self-review) trace back to Superpowers. Craft keeps those and drops the paperwork.

## Upstream sync (for maintainers)

Visual Companion scripts in `skills/brainstorm/scripts/` are vendored from `superpowers:brainstorming`. See `CREDITS.md` for full attribution. To check for upstream improvements:

```bash
diff -r ~/.claude/plugins/cache/claude-plugins-official/superpowers/<latest>/skills/brainstorming/ ./skills/brainstorm/
```

Read the diff, port selectively.

## License

Personal workflow skills. Use, adapt, ignore - whatever fits your flow.
