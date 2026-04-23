# <Feature Name> - Snapshot (YYYY-MM-DD)

**Status:** WIP | shipping | stable
**Owner:** <name>
**Supersedes:** snapshots/YYYY-MM-DD.md (or "first snapshot")

## Goal
One paragraph on what this feature is and why it exists.

## Branch / Merge Status

The three commands that produced this map — include them verbatim so the next session can reproduce:
- `git rev-parse --abbrev-ref HEAD`
- `git log --oneline <main>..HEAD`
- `git log --since="<previous-snapshot-date>" --oneline <main> -- <feature paths>`

### Landed on main since <previous-snapshot-date>
- `<sha>` **<PR #>** <title> - one-line summary.

### On the active branch (`<branch>`, N commits ahead of main)
- `<sha>` **<subject>** (<date>) - one-line summary.

### Still elsewhere / still WIP
- Anything called out in the prior snapshot that hasn't moved (other branches, external PRs). Be explicit about unchanged state.

### Implications for this snapshot
- One or two bullets on how the merge state reshapes the rest of this document: which sections move from WIP to shipped, which pre-wired assumptions still apply, what the next merge/rebase step should be.

Omit this section entirely on a first snapshot with no prior merge history (e.g. design-only features).

## Current Architecture
2-4 paragraphs. Describe the shape: main components, how they collaborate, critical boundaries. Pull from code, not from old specs. No code dumps.

## Key Files
- path/to/module.<ext> - responsibility
- path/to/other.<ext> - responsibility

## Decisions Made (and rationale)
- Chose A over B because ...
- Deferred C because ...

## Known Gaps / Open Questions
- Not yet built: ...
- Stubbed: ...
- Next session should tackle: ...

## Load This In Next Session
To get full context without re-reading specs, load:
- This file
- features/<slug>/overview.md (if it exists)
- 2-3 most critical implementation files - the ones a new session would need open to make good decisions.

Omit this section entirely on a first snapshot if there are no meaningful implementation files yet (design-only feature).

## Related Specs
- specs/YYYY-MM-DD-<topic>-design.md - still relevant: <which parts>
- specs/YYYY-MM-DD-<topic>-design.md - superseded
