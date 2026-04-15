# Changelog

All notable changes to this plugin are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

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
