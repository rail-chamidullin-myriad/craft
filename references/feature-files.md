# Feature Files — Roles, Read Protocol, Write Protocol

Per-feature memory lives under `<memory-root>/features/<slug>/`. The layout itself is defined in [`memory.md`](memory.md); this file defines what each file *means* and how every craft skill should read from and write to them. Read this before any skill opens a session on an existing feature or writes to feature memory.

`<memory-root>` is defined in `memory.md`.

## Files

### `overview.md` — the always-loaded anchor

Small, mutable, semantically two parts:

1. **Canonical sections** (Product / Business, Architectural Invariants, External Constraints) - long-lived decisions that outlive any single implementation. Author-only: skills propose, the user confirms, only `craft:implement` and `craft:feature-state` write here.
2. **Recent Changes index** - one line per recent memorable implement session, newest top, FIFO trimmed to the last 10. Mechanically managed by `craft:implement` at finish.

Plus optional `Open Questions` and a `History` log of dated additions to the canonical sections.

Hard cap: **5k chars (~1.2k tokens).** This file is loaded by every craft session on the feature; bloat taxes every future load.

Author rules, three buckets, filter, FIFO@10 spec, mutable-with-consolidation rule, and template: [`overview.md`](overview.md).

### `changes/YYYY-MM-DD-<short-topic>.md` — per-session leaf files

Each file captures one implement session's delta in 1-2 paragraphs: what shipped, anything non-obvious a reader would miss from `git log` + code, optional gaps left open. By construction it cannot grow into a state-of-the-feature dump - it only describes one session.

Hard cap: **2k chars (~500 tokens).**

Loaded on demand only - when the `Recent Changes` index entry or the current task references a specific changes file. Never default-loaded.

Template: [`templates/changes.md`](templates/changes.md).

### `snapshots/YYYY-MM-DD.md` — DEPRECATED

The previous dated state-snapshot artifact. No new snapshots are written; existing files remain on disk as legacy reference. Do not edit, do not delete, do not migrate. Loaders never default-read them. If a user needs a frozen full-state distillation today, the answer is `craft:feature-state` consolidating `overview.md`, not a new snapshot.

## Read protocol — loading prior context on an existing feature

Any skill starting work on an existing feature (brainstorm, implement, feature-state) should:

1. Check if `<memory-root>/features/<slug>/` exists. If not, it's a new feature - skip.
2. Read `overview.md` in full. The canonical sections are not re-litigated unless the user flags them as stale; the Recent Changes index points at leaf files.
3. Read `changes/YYYY-MM-DD-*.md` **on demand only** - when an index entry or the current task plainly references one. Do not bulk-read the directory.
4. Do NOT default-read prior specs in `<memory-root>/specs/`. Only open a spec if `overview.md` or a relevant changes file points at an unresolved question that lives there.
5. Do NOT default-read legacy `snapshots/*.md`. Open one only if a current task explicitly references it as historical context.
6. Restate what you loaded to the user before proceeding. Ask whether any of it is outdated.

## Size budgets (caveman)

- `overview.md` ≤ 5k chars (~1.2k tokens). Loaded by every session that touches the feature.
- each `changes/*.md` ≤ 2k chars (~500 tokens). Loaded on demand.

If a read step finds `overview.md` over budget, flag it during the restate step and suggest invoking `craft:feature-state` to compact. A `changes/*.md` over budget means scope was wrong - the file is trying to be a snapshot. Flag it but do not auto-fix.

## Write protocol — `overview.md`

Mutable with consolidation discipline. Both `craft:implement` (at finish) and `craft:feature-state` (during compaction) may write here. Rules:

- **Check for duplicates before adding** (paraphrase match, not just exact string). Do not write a near-duplicate of an existing entry.
- **Resolve contradictions.** When a new decision overrides an old one, replace the old line; do not accumulate two contradicting entries.
- **Consolidate near-duplicates** that drifted into different wordings during prior sessions.
- **Recent Changes index** is FIFO managed: append the new entry at the top, trim to the last 10.
- **Canonical sections** (Product / Architectural / External) are author-only: only added, edited, or removed on explicit user confirmation. Adds get a dated `History` line.

Full rules: [`overview.md`](overview.md).

## Write protocol — `changes/*.md`

Only `craft:implement` writes here, at session finish:

1. Decide if the session is memorable. If a future reader would learn nothing beyond what `git log` + code already say, skip the write entirely. The default for routine sessions is no file.
2. `mkdir -p <memory-root>/features/<slug>/changes/`.
3. Draft the file from session context (TodoWrite log, spec, conversation). Keep under 2k chars.
4. Write to `<memory-root>/features/<slug>/changes/YYYY-MM-DD-<short-topic>.md`. Same-day collision: append `b`, `c`, etc.
5. Append the index entry to `overview.md` Recent Changes (newest top), trim to the last 10.
6. Never edit or delete an existing changes file - new facts go in a new dated entry.

## Invariants

- **`overview.md` is mutable with consolidation discipline.** No duplicates; resolve contradictions; trim Recent Changes to last 10.
- **`changes/*.md` are immutable once written.** Author a new dated file to correct a past one; do not edit in place.
- **`snapshots/*.md` are deprecated.** Read-only legacy; never default-loaded; never written to.
- **Code is the source of truth.** When memory disagrees with current code, fix memory via consolidation or a new changes file; do not edit old files to "fix" them.
