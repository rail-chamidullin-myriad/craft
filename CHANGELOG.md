# Changelog

All notable changes to this plugin are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [0.6.0] - 2026-04-23

### Added
- `craft:feature-state` - canonical three-command branch/merge probe in step 3 (read code + history): `git rev-parse --abbrev-ref HEAD`, `git log --oneline <main>..HEAD`, and `git log --since="<last-snapshot-date>" --oneline <main> -- <feature-paths>`. Gives snapshots a clear shipped-vs-WIP picture before diving into commit bodies. Optional `gh pr list`/`gh pr view` augmentation with explicit fallback guidance so TLS/auth failures don't block the snapshot.
- `references/templates/snapshot.md` - new **Branch / Merge Status** section capturing the three commands verbatim plus buckets for "Landed on main since <previous-snapshot-date>", "On the active branch", "Still elsewhere / still WIP", and "Implications for this snapshot". Omitted on first snapshots with no prior merge history.

## [0.5.0] - 2026-04-22

### Changed (breaking)
- **Feature state layout simplified to `snapshots/YYYY-MM-DD.md`.** The previous pile of dated `YYYY-MM-DD-<slug>-state.md` files directly under `features/<slug>/` is replaced with a single `snapshots/` directory holding dated, immutable snapshots. **Newest filename wins** — no separate "current" pointer file. (A `state.md` + `snapshots/` variant was considered and rejected: it duplicated the latest snapshot's content at a stable path, adding a second write per update and a drift risk, without enough reader-side benefit to justify it.)
- **Task logs removed.** `craft:implement` no longer writes a dated task log at finish, and the `tasks/` directory is no longer part of the memory layout. Session traceability is carried by commits + TodoWrite + the (optional) feature-state refresh instead.

### Added
- `references/feature-files.md` - new shared reference defining the role of `overview.md` and `snapshots/*.md`, the read protocol (load newest snapshot + overview), and the write protocol (append a new snapshot; never edit or delete). Linked from all three SKILL.md files so feature-file handling stays consistent without per-skill duplication.

### Changed
- `references/overview.md` - moved up from `skills/feature-state/references/overview-md.md` to `references/overview.md` (shorter name; unambiguous since it lives under `references/`). Keeps the guidance + inline template as one file — the template was too small and too specific to justify splitting into its own file.
- Templates consolidated under `references/templates/`: `spec.md` (was `skills/brainstorm/templates/`) and `snapshot.md` (was `skills/feature-state/templates/`). Cross-links in the brainstorm and feature-state skills updated.
- `craft:feature-state` - auto-detect and update-mode logic now keys off "any file in `snapshots/`". On update, write a new dated snapshot (no archive-before-overwrite step). Template renamed from `templates/state-file.md` to `templates/snapshot.md`.
- `craft:brainstorm` step 1 ("load prior context") condensed to a pointer at `references/feature-files.md` — the read-protocol prose now lives in one place instead of being duplicated across skills.
- `references/memory.md` - layout and invariants updated for the snapshots-only model; file-role and read/write protocol details delegated to `feature-files.md`.
- README layout block updated to match.

### Removed
- `skills/implement/templates/task-log.md` - no longer needed.
- Duplicated "load prior feature context" prose in `brainstorm` step 1 and `feature-state` step 2 (both now reference `feature-files.md`).

## [0.4.0] - 2026-04-22

### Changed (breaking)
- **Memory location moved out of the working tree.** Craft memory (specs, task logs, feature state) now lives at `$HOME/.claude/projects/<project-slug>/craft/memory/` — derived at runtime from the project root. Previously defaulted to `docs/craft/` at the repo root with an optional `CLAUDE.md` override. Motivation: git worktrees each have their own working tree, so in-tree memory doesn't share across worktrees and pollutes `git status`. The new location is a sibling to Claude Code's built-in auto-memory (`~/.claude/projects/<slug>/memory/`), is naturally shared across all worktrees, and never shows up in `git status`.
- **Override removed.** There is no longer a way to override the memory root. One canonical path, derived the same way for every project. Users who had a craft redirect in their `CLAUDE.md` / `CLAUDE.local.md` should remove it. (YAGNI — the override existed to let users redirect away from `docs/craft/`, which the new default makes unnecessary.)
- **Migration for existing users:** move the contents of the old in-tree `docs/craft/` (or whatever override you had) to `~/.claude/projects/<project-slug>/craft/memory/`. `<project-slug>` is the absolute repo path with `/` replaced by `-` (same encoding Claude Code uses for session logs).

### Added
- `references/memory.md` - shared reference defining the memory path, layout, and invariants. All three skills delegate to this single file so path/layout conventions stay consistent.

### Changed
- `craft:brainstorm` step 9 (post-write) - no longer asks the user to open and review the written spec. The spec is now typically outside the working tree and awkward to browse, and the user already approved each section during step 5. The skill now sends a 3-5 bullet recap of key decisions and asks for corrections instead of a file-review ask.
- All three SKILL.md descriptions rewritten to include explicit trigger phrases per Claude Code skill-authoring guidance (descriptions drive skill invocation, so concrete trigger contexts help Claude consult the right skill). Moved the body `## When to use` sections into the description and removed them from the body.

### Removed
- `## When to use` body sections from all three skills (content folded into descriptions per above).
- All references to "Default artifact root" and CLAUDE.md override in SKILL.md files and README.

## [0.3.0] - 2026-04-15

### Changed
- `craft:implement` - added an explicit "surgical changes only" rule to the execution loop and two matching Red Flag rows. Inspired by [Andrej Karpathy's observations on LLM coding](https://x.com/karpathy/status/2015883857489522876) - in particular, that agents tend to "improve" adjacent code, drift from existing style, and remove pre-existing code they don't fully understand.
- `craft:feature-state` - SKILL.md reduced 214 → 108 lines (-50% always-loaded context). Collapsed three sequential "ask the user" gates into a single batched first-message pattern with an explicit template, since benchmarking showed prose requests for "one round-trip" don't override Claude's default step-by-step asking — a concrete example does. Extracted the state-file template to `templates/state-file.md` and the `overview.md` guidance + template to `references/overview-md.md` so they're loaded only when needed. Verified against a real feature (`llm-file-upload`) with 3 eval scenarios x 2 skill versions; no quality regression.
- `craft:brainstorm` - SKILL.md reduced 195 → 133 lines (-32%). Removed the Checklist section (duplicated "The Process" headings) and the Key Principles section (every bullet already appeared in the body). Compressed the inline Visual Companion section since `visual-companion.md` already holds the detail. Extracted the spec template to `templates/spec.md`.
- `craft:implement` - SKILL.md reduced 258 → 218 lines (-16%). Extracted the subagent-dispatch protocol (criteria, status-report handling, model selection) to `references/subagent-dispatch.md`, since it's a meta-concern that only applies on the exception path. Extracted the task-log template to `templates/task-log.md`.

## [0.2.0] - 2026-04-14

### Changed
- Generalized all skills to be project-agnostic: removed hard-coded `just` shortcuts, `.py` file extensions, and references to `.claude/rules/` and `CLAUDE.local.md`. Skills now defer to each project's `CLAUDE.md` for conventions.
- `craft:brainstorm` - reworked Visual Companion as an opt-in tool with a one-time consent prompt; added explicit guidance on when to ask questions; dropped the embedded "Upstream Sync" section (README already covers it).
- `craft:implement` - expanded worktree setup with explicit `.worktrees/` directory selection, gitignore verification, project setup detection, and clean-baseline check; added a scope heuristic for when to switch from-conversation mode back to brainstorm; replaced the hand-off to `superpowers:finishing-a-development-branch` with a neutral 3-option worktree hand-off menu; generalized verification commands.
- `craft:feature-state` - auto-detects full vs. update mode based on prior state file; expanded discovery to include `docs/craft/tasks/` task logs; tightened `overview.md` filter to three buckets (product/business, architectural invariants, external constraints) with explicit exclusions.
- Visual Companion scripts now persist session data under `.craft/brainstorm/` instead of `.superpowers/brainstorm/`.
- README - clarified positioning relative to Superpowers; documented `.worktrees/` and `.craft/brainstorm/` as gitignored directories.

## [0.1.0] - 2026-04-13

### Added
- Initial release.
- `craft:brainstorm` skill - spec-driven brainstorming with hole-finding discipline and optional Visual Companion.
- `craft:implement` skill - in-session spec execution with worktree isolation, TodoWrite ledger, TDD cadence, and dated task logs.
- `craft:feature-state` skill - dated feature snapshots and an evolving `overview.md` anchor per feature.
- Visual Companion scripts vendored from `superpowers:brainstorming` for browser-based mockups and diagrams.
- README with installation instructions (local marketplace, symlink, published repo).
- CREDITS documenting attribution to superpowers.
