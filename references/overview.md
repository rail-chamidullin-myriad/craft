# overview.md guidance

`overview.md` is different from a snapshot. It is a small, stable list of decisions that outlive any given implementation. Not dated — it evolves in place.

**Hard cap: 5k chars (~1.2k tokens).** This file is loaded by every craft session that touches the feature. Bloat here taxes every future load.

## The filter (one line)

If a future reader could answer this by reading the code, it does NOT belong in `overview.md`.

## Format rule (caveman)

**Each invariant is ONE LINE, ≤ 200 chars, plus a Source citation on the same line or next line.** No multi-paragraph explanations, no "do not do X" warnings, no rationale prose. The invariant statement should be self-evidently load-bearing; if it needs explaining, it belongs in a snapshot's `Decisions Made` section, not here.

The History section: one line per dated session, comma-separated bullet of what was added.

## Three buckets — only log entries that fit one of these

1. **Product / business** - why the feature exists, who it serves, what counts as done, explicit scope boundaries. Recoverable from stakeholder conversations, not from code.
   - Example: "This feature exists to satisfy the EU battery passport requirement - not as a generic audit log."
2. **Architectural invariants** - boundaries, ownership, cross-cutting patterns that constrain future decisions.
   - Example: "Module A must never call Module B directly - all traffic goes through the event bus."
   - Example: "All new features in this domain must gate behind a feature-flag entry."
3. **External constraints** - compliance, legal, SLAs, contracts with other teams or systems.
   - Example: "Response time must stay under 200ms; agreed with the mobile team for their cache TTL."

## Explicitly excluded (redirect to the right place)

- Function names, file paths, signatures, type shapes → read the code.
- Naming or refactor choices ("renamed X to Y") → git history.
- Implementation mechanics ("uses a dict keyed by user_id") → code.
- Session-level "we deferred X" → that's what the latest snapshot's *Known Gaps* section is for.
- Rejected alternatives, unless the rationale is load-bearing for future decisions.

## Template (only if creating overview.md fresh)

```markdown
# <Feature Name> - Overview

Long-lived decisions for <feature>. Short, stable, evolves in place. Every entry traces back to a spec or a session, and fits one of: product/business, architectural invariant, or external constraint.

## Product / Business
- <Decision>. Source: specs/YYYY-MM-DD-<topic>-design.md

## Architectural Invariants
- <Invariant>. Source: specs/YYYY-MM-DD-<topic>-design.md

## External Constraints
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md or external doc.

## Open Questions
- <Question>.

## History
- YYYY-MM-DD: added <decision> under <bucket> (source: spec or session).
```

## Appending to existing overview.md

Put each new entry under the correct bucket and add a `History` line. Do NOT rewrite existing entries unless the user explicitly asks.

When appending, also check whether the file is approaching the 5k-char cap. If yes, look for entries that have become stale (an invariant the codebase no longer follows, a constraint that has been removed) and propose deletions to the user. The file decays into noise if it never sheds entries.
