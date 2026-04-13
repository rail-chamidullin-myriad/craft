# Craft

Disciplined, spec-driven software development skills for Claude Code.

Craft started as a few small edits to Superpowers and turned into its own thing. Two observations drove it.

First, the implementation plan file wasn't earning its keep. It existed to hand work off to a fresh subagent, but most of my implementation work fits better in the main session - full context, fewer translation layers, faster to course-correct. Once the main session does the work, the plan file is just a restatement of the spec. So craft drops it. TodoWrite carries the task ledger, subagents come in only for isolated, mechanical tasks that don't touch architecture.

Second, anything non-trivial takes more than one session. You brainstorm, you ship a piece, you come back next week with a new question and a new spec. By spec five, each session spends a chunk of its context re-deriving what was already decided. Craft adds per-feature artifacts for this: a dated state snapshot after meaningful changes, and an `overview.md` that accumulates canonical decisions. The next brainstorming session starts from one file, not the whole spec pile, and each new spec builds on the last.

Everything else - the spec template, the TDD cadence, the Visual Companion - comes from Superpowers and stays close to the original.

## Skills

- **`craft:brainstorm`** - Collaborative, spec-driven brainstorming. Loads prior feature state, asks clarifying questions one at a time (with a recommended answer where useful), proposes 2-3 approaches with trade-offs, and presents the design in sections for incremental approval. Optional browser Visual Companion for UI mockups, diagrams, and visual A/B selection. Produces a dated spec.
- **`craft:implement`** - Execute a feature in the current session. Two modes: *from-spec* (read a design spec from disk and implement it) or *from-conversation* (implement directly from the current chat, no spec file - for when you already know exactly what to do). Uses a git worktree for isolation, TodoWrite as the task ledger, and a TDD cadence (failing test → run → impl → run → commit). Writes a dated task log at finish. Optionally prompts to update feature state and overview.
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

Override the artifact root in your project's `CLAUDE.md` or `CLAUDE.local.md` if you want artifacts somewhere else (e.g. a personal subdirectory).

## Installation

### Option A: marketplace install from a local path (recommended)

From your Claude Code session in the repo:

```
/plugin marketplace add ./.claude/plugins/craft
/plugin install craft@craft
```

The marketplace is named `craft` (see `.claude-plugin/marketplace.json`) and it contains one plugin also named `craft` - hence `craft@craft`. If the marketplace command fails, skip to Option B.

### Option B: symlink into the global plugin cache

```bash
# Adjust paths to your checkout location.
ln -s "$(pwd)/.claude/plugins/craft" ~/.claude/plugins/cache/local-craft

# Register in installed_plugins.json (manual edit) or reopen Claude Code
# and check /plugin list.
```

### Option C: fork this plugin directory into its own repo

Once you're happy with the skills, split `.claude/plugins/craft/` into its own git repo and install as a normal remote plugin:

```
/plugin marketplace add <your-user>/craft-claude-skills
/plugin install craft
```

## Why not just use superpowers?

Use superpowers. Craft is a thin opinionation layer on top for my workflow:

| Superpowers | Craft |
|-------------|-------|
| `brainstorming` → spec on disk | `craft:brainstorm` → spec on disk (same shape, adds feature-state preload and spec self-check; hands off to `craft:implement` instead of `writing-plans`) |
| `writing-plans` → plan file on disk | (skipped entirely) |
| `executing-plans` / `subagent-driven-development` → plan re-read in new session, subagent per task | `craft:implement` (from-spec) → read spec directly, main session loop, TodoWrite ledger |
| No shortcut for "I know exactly what to do, just implement it" | `craft:implement` (from-conversation) → restate agreed scope inline, confirm, execute; no spec file needed |
| No feature-state concept | `craft:feature-state` + `features/<slug>/overview.md` to bound context across large multi-spec features |
| `finishing-a-development-branch` | (skipped - `craft:implement` ends with verify + task log) |

The engine ideas (one-question-at-a-time questioning, propose 2-3 approaches, present design in sections, TDD cadence, file-structure pre-flight, no-placeholder discipline, spec self-review) come from superpowers. Craft keeps those, removes the paperwork.

## Upstream sync

Visual Companion scripts in `skills/brainstorm/scripts/` are vendored from `superpowers:brainstorming`. To check for upstream improvements:

```bash
diff -r ~/.claude/plugins/cache/claude-plugins-official/superpowers/<latest>/skills/brainstorming/ .claude/plugins/craft/skills/brainstorm/
```

Read the diff, port selectively.

## License

Personal workflow skills. Use, adapt, ignore - whatever fits your flow.
