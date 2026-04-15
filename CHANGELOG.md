# Changelog

All notable changes to this plugin are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

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
