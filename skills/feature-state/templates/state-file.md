# <Feature Name> - State (YYYY-MM-DD)

**Status:** WIP | shipping | stable
**Owner:** <name>
**Supersedes:** YYYY-MM-DD-<slug>-state.md (or "first state file")

## Goal
One paragraph on what this feature is and why it exists.

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

Omit this section entirely on a first state file if there are no meaningful implementation files yet (design-only feature).

## Related Specs
- specs/YYYY-MM-DD-<topic>-design.md - still relevant: <which parts>
- specs/YYYY-MM-DD-<topic>-design.md - superseded
