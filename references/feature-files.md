# Feature Files — Roles, Read Protocol, Write Protocol

Per-feature memory lives under `<memory-root>/features/<slug>/`. The layout itself is defined in [`memory.md`](memory.md); this file defines what each file *means* and how every craft skill should read from and write to them. Read this before any skill opens a session on an existing feature or writes a new snapshot.

`<memory-root>` is defined in `memory.md`.

## Files

### `overview.md` — canonical decisions (append-only, stable)

Small, long-lived list of decisions that outlive any single implementation: product/business rules, architectural invariants, and external constraints. Not dated — it evolves in place. Only appended to on explicit user confirmation.

Author rules, three buckets, filter, and template: [`overview.md`](overview.md).

### `snapshots/YYYY-MM-DD.md` — current state (dated, immutable)

Each file captures the feature's state at one point in time: goal, current architecture, key files, decisions made, known gaps, related specs. The **newest by filename** is authoritative; older ones are the audit trail.

Template: [`templates/snapshot.md`](templates/snapshot.md).

## Read protocol — loading prior context on an existing feature

Any skill starting work on an existing feature (brainstorm, implement, feature-state re-distill) should:

1. Check if `<memory-root>/features/<slug>/` exists. If not, it's a new feature — skip.
2. If `overview.md` exists, read it in full. Canonical decisions — do not re-litigate unless the user flags them as stale.
3. If `snapshots/` contains any files, list them and read only the newest (`ls | sort | tail -1` — `YYYY-MM-DD` sorts lexicographically). That file is authoritative. Skip older snapshots unless investigating a specific past decision.
4. Do NOT default-read prior specs in `<memory-root>/specs/`. Only open a spec if the snapshot or overview points at an unresolved question that lives there.
5. Restate what you loaded to the user before proceeding. Ask whether any of it is outdated.

## Write protocol — new snapshot (feature-state only)

1. `mkdir -p <memory-root>/features/<slug>/snapshots/`.
2. Find the previous snapshot filename (newest in `snapshots/`), if any. Use it for the `Supersedes:` field.
3. Write the new file to `<memory-root>/features/<slug>/snapshots/YYYY-MM-DD.md` using today's date, following the template.
4. Never edit or delete an existing snapshot. If two snapshots land on the same day, append a suffix (e.g. `2026-04-22b.md`) — existing files are immutable.

## Write protocol — appending to `overview.md`

Only on explicit user yes. Place the new entry under the correct bucket (product/business, architectural invariant, external constraint) and add a `History` line. Do not mass-rewrite existing entries. Full rules: [`overview.md`](overview.md).

## Invariants

- **Newest snapshot wins.** No separate "current" pointer file. Filename date is the ordering.
- **Snapshots are immutable.** Author a new one to correct a past one; don't edit in place.
- **`overview.md` is append-only.** Stable, canonical, never mass-rewritten.
- **Code is the source of truth.** When a snapshot or overview disagrees with current code, re-distill via `craft:feature-state`. Do not edit the old snapshot to "fix" it.
