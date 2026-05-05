# Changelog

All notable changes to this plugin are documented here. Format loosely follows [Keep a Changelog](https://keepachangelog.com/).

## [0.9.0] - 2026-05-05

### Changed
- **`craft:brainstorm` now writes feature memory at finish.** Step 10 expanded from "offer overview update and hand off" to a four-part finish: (1) auto-create `<memory-root>/features/<slug>/` with an empty `overview.md` if missing; (2) append a `Recent Changes` index entry pointing at the new spec file (`- YYYY-MM-DD: <topic> -> specs/YYYY-MM-DD-<topic>-design.md`), FIFO trim to 10; (3) propose canonical-decision diffs (unchanged); (4) bloat-flag if `overview.md` exceeds 5k chars and suggest `craft:feature-state`. Brainstorm still does NOT write `changes/*.md` - the spec under `<memory-root>/specs/` is the brainstorm session's record. Motivation: `Recent Changes` was previously implement-only, so brainstorm sessions silently dropped out of the rolling timeline; users had to manually maintain a `History` line to track them. Now `Recent Changes` is a unified per-feature timeline where each entry points at either a spec (brainstorm) or a changes file (implement), with no duplication.
- **`Recent Changes` index format extended to allow spec-pointing entries.** The index spec in `references/overview.md` and `references/feature-files.md` now documents two valid line formats: brainstorm entries point at `specs/...-design.md`, implement entries point at `changes/...md`. Both share the same FIFO@10 budget. The `changes/*.md` write contract is unchanged - only `craft:implement` writes those files.
- `references/overview.md` - "Mutable-with-consolidation rule" section retitled and updated to list all three skills (`craft:brainstorm`, `craft:implement`, `craft:feature-state`) as legitimate writers of `overview.md`.
- `references/feature-files.md` - canonical-section author list and write protocol updated to include `craft:brainstorm` alongside `craft:implement` and `craft:feature-state`.

## [0.8.0] - 2026-05-03

### Changed (breaking)
- **Per-feature memory layout switched from snapshots to `overview.md` + `changes/`.** New layout is one always-loaded `overview.md` (canonical sections + a FIFO@10 `Recent Changes` index) plus per-session leaf files under `changes/YYYY-MM-DD-<short-topic>.md` (1-2 paragraphs, 2k-char cap, loaded on demand). The dated `snapshots/YYYY-MM-DD.md` artifact is deprecated and read-only - no new snapshots are written. Existing snapshot files remain on disk as legacy reference; no migration. Motivation: real-world snapshots had crept to 27k chars (vs. an 8k cap), so users ran `craft:feature-state` rarely and memory drifted. Discipline now comes from scope (a changes file describes one session's delta and cannot grow into a full state dump) instead of a size cap on a panoramic artifact. Modeled on Claude Code auto-memory's index + leaves pattern.
- **`overview.md` is mutable with consolidation discipline, not append-only.** Both `craft:implement` (at finish) and `craft:feature-state` (during compaction) may add, edit, merge, or remove entries when they spot duplicates or contradictions. Same auto-memory rule: do not write duplicates; update or remove outdated entries. Canonical-section adds and removals each get a `History` line.
- **`craft:feature-state` reshaped around compaction, not snapshotting.** The skill no longer writes dated `snapshots/*.md`, no longer runs the canonical three-command branch/merge probe, and no longer follows the "delta-only after the first snapshot" ruleset. New responsibilities: consolidate canonical sections in `overview.md`, propose promotions from `changes/*.md` to canonical, trim `Recent Changes` past 10, and flag bloat over the 5k-char cap. Invoked only on user request or when a loader trips a budget warning.
- **`craft:implement` auto-creates feature memory and writes per-session memory at finish.** New step 1 resolves a feature slug (from spec topic, branch name, or existing-features near-match), creates `<memory-root>/features/<slug>/` with an empty `overview.md` if missing, and asks once when the slug is genuinely ambiguous. Rewritten step 9 decides whether the session is memorable (skip if recoverable from `git log` + code alone), drafts a 1-2 paragraph `changes/*.md` from session context, appends an entry to the `Recent Changes` index (FIFO trim to 10), and proposes any canonical-decision diffs surfaced by the session. The standalone "update feature-state?" prompt is removed.

### Added
- `references/templates/changes.md` - new per-session leaf template with header (Branch, Spec), `What shipped`, `Gaps left open`, 2k-char hard cap, and immutability rule.
- `references/overview.md` - new `Recent Changes` section spec (FIFO@10, format `- YYYY-MM-DD: <topic> -> changes/<file>.md`), index budget math (~1.2k for index, ~3.8k for canonical sections under the 5k cap), and the shared mutable-with-consolidation rule used by both `craft:implement` and `craft:feature-state`.

### Removed
- `references/templates/snapshot.md` - the dated state-snapshot template. New writes use `references/templates/changes.md`; existing `snapshots/*.md` on disk are kept untouched as legacy reference.
- `craft:feature-state` - the batched first-message protocol (slug + scope + file list + spec list), the canonical three-command branch/merge probe, the `Branch / Merge Status` snapshot section, the "delta-only after the first snapshot" ruleset, and dated-snapshot writes.
- `craft:implement` - the standalone "update feature-state?" prompt at finish; replaced by automatic `changes/*.md` write + Recent Changes index trim + proposed canonical-decision diff.
- `craft:brainstorm` - the "newest snapshot" line in the read protocol; the standalone feature-state hand-off prompt at finish (the canonical-decisions append survives).

## [0.7.0] - 2026-04-28

### Changed
- **Snapshots are now delta-only after the first one.** `references/templates/snapshot.md` and `skills/feature-state/SKILL.md` rewritten so each new snapshot only documents what changed since the prior snapshot. Sections that haven't moved read "Unchanged from `snapshots/YYYY-MM-DD.md`." instead of restating prior content. `Branch / Merge Status` is one line per commit (`<sha> <subject>`) with body prose only when a commit shifted an architectural invariant. `Key Files` and `Decisions Made` list only what materially shifted; older inventory stays in the prior snapshot.
- `references/templates/snapshot.md` - `Goal` and `Current Architecture` sections explicitly allow "Unchanged from prior snapshot" as the entire body; `Current Architecture` capped at 1-2 paragraphs (was 2-4); `Related Specs` skipped entirely if nothing moved.

### Added
- **Hard size budgets** in `references/feature-files.md`, `references/overview.md`, and `skills/feature-state/SKILL.md`: `overview.md` ≤ 5k chars (~1.2k tokens), newest snapshot ≤ 8k chars (~2k tokens). Rationale: a snapshot only earns its keep if loading it is cheaper than reading the code; budgets force compression instead of accumulation. Read protocol now flags over-budget files during the restate step and suggests re-distillation.
- `references/overview.md` - **one-line-per-invariant format rule** (≤ 200 chars + Source citation). Multi-paragraph explanations and rationale prose belong in a snapshot's `Decisions Made`, not the overview. Append guidance now also instructs proposing deletions of stale entries when the file approaches the 5k cap.
- `skills/feature-state/SKILL.md` - four new Red Flag rows covering the most common bloat traps: "Carried from prior snapshot…" sections, multi-sentence commit summaries, re-listing every key file each snapshot, and "the snapshot is at 15k chars but it's all useful".

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
