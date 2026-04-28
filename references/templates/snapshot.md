# <Feature Name> - Snapshot (YYYY-MM-DD)

**Status:** WIP | shipping | stable
**Owner:** <name>
**Supersedes:** snapshots/YYYY-MM-DD.md (or "first snapshot")

**Hard cap: 8k chars total (~2k tokens). Above that, reading the code is cheaper than loading this file - which defeats the snapshot's purpose.**

## Goal
One short paragraph. Skip if unchanged from prior snapshot - just write "Unchanged from `snapshots/YYYY-MM-DD.md`."

## Branch / Merge Status

Run these and record the output. **One line per commit: `<sha> <subject>`. No per-commit prose unless the commit shifted an architectural invariant.**
- `git rev-parse --abbrev-ref HEAD`
- `git log --oneline <main>..HEAD`
- `git log --since="<previous-snapshot-date>" --oneline <main> -- <feature paths>`

### Landed on main since <previous-snapshot-date>
- `<sha>` <subject>

### New on the active branch since <previous-snapshot-date>
- `<sha>` <subject>

Older commits on the branch already appeared in the prior snapshot - do not restate. The reader can `git log` for full history.

### Implications for this snapshot
One or two bullets only. Skip the section if there are no implications worth flagging.

Omit this entire section on a first snapshot with no prior merge history.

## Current Architecture

**1-2 paragraphs MAX.** Describe ONLY what is new or changed since the prior snapshot. If the architecture is unchanged, the entire section reads "Unchanged from `snapshots/YYYY-MM-DD.md`." and that's it. The first snapshot is the only one where you describe the architecture in full.

No code dumps. No restating prior architecture in different words.

## Key Files

Only files that materially shifted in this snapshot. Cite the prior snapshot for the full inventory.

- path/to/module.<ext> - what shifted

If nothing shifted, write "Unchanged from `snapshots/YYYY-MM-DD.md`." and skip the bullets.

## Decisions Made

Only NEW decisions in this snapshot, with one-sentence rationale each. Do not reproduce unchanged decisions from prior snapshots - they are still in force unless explicitly reversed here.

- Decision X. Why: <one short sentence>.

## Known Gaps / Open Questions

Only NEW or CHANGED gaps. For unchanged gaps, write "See `snapshots/YYYY-MM-DD.md`." Do not reproduce the prior list.

- Gap X.
- Resolved since prior: ~~old gap~~ (if it shifts state explicitly).

## Load This In Next Session

To get full context without re-reading specs:
- This file
- features/<slug>/overview.md (if it exists)
- 1-3 critical implementation files

Omit this section entirely on a design-only first snapshot.

## Related Specs

Only specs whose status changed. Skip the section if nothing moved.
- specs/YYYY-MM-DD-<topic>-design.md - <still relevant: which parts | superseded>
