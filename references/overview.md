# overview.md guidance

`overview.md` is the always-loaded anchor for a feature. Small, mutable, two parts:

1. **Canonical sections** - long-lived constraints (product/business, architectural invariants, external constraints) that outlive any single implementation. Author-only, evolved with consolidation discipline.
2. **Recent Changes index** - one line per recent memorable session (brainstorm or implement), newest top, FIFO trimmed to the last 10. Mechanically managed by `craft:brainstorm` (pointing at a spec file) and `craft:implement` (pointing at a changes file).

That's it. No `Open Questions`, no `Recommended Next Steps`, no `History`. Anything that isn't a current constraint or a Recent Changes index entry does not belong here.

**Hard cap: 5k chars (~1.2k tokens).** This file is loaded by every craft session on the feature. Bloat taxes every future load.

## The filter (two lines)

1. **If a future reader could answer it by reading the code, it does NOT belong here.**
2. **If the line *describes current code* instead of *constraining future code*, it does NOT belong here.**

The second filter is the one most often missed - see the constraint-vs-description test below.

## Canonical entry format (caveman)

**One atomic fact per line, ≤ 200 chars including citation, stated as a constraint not a description.**

- **Atomic.** One fact per line. Multi-clause sentences bundle 3 facts that decay independently - one rotting clause invalidates the whole line.
- **Constraint, not description.** Write what must / must-not hold, not what currently exists. *"Regulations are selected per-message, never chat-level"* is a constraint. *"Regulations are stored in `AssistantMessageV2SettingsRecord` (JSONB)"* is a description - the moment storage changes, the line lies.
- **Citation is a pointer, not authority.** `Source: specs/<file>.md` or `Source: session YYYY-MM-DD`. Lets a human find the conversation that produced the rule. Code is the source of truth for whether the rule still holds; do not let citations grow into provenance prose ("PR #4836 review from Jan Plesnik, commit `d37ebdba4`").

**Constraint-vs-description stress test (apply before adding or keeping a line):**

> "If the implementation flipped tomorrow - different storage, different shape, different module - would this line need rewriting?
>
> Yes → it's a description. Drop or reword to the underlying constraint.
> No → it's a constraint. Keep."

## Recent Changes section spec

- Newest entry at the top. FIFO trim to the last 10.
- One line per entry. Format depends on the writing skill:
  - `craft:brainstorm` at finish: `- YYYY-MM-DD: <short-topic> -> specs/YYYY-MM-DD-<short-topic>-design.md`
  - `craft:implement` at finish: `- YYYY-MM-DD: <short-topic> -> changes/YYYY-MM-DD-<short-topic>.md`
- Both skills append mechanically: at the top, trim to 10. No judgment.
- Spec files live under `<memory-root>/specs/`; changes files live under `<memory-root>/features/<slug>/changes/`. Both load on demand only.
- Older entries (past the 10th most recent) stay on disk as audit trail; they are simply not pointed at from this index.
- Index budget: 10 lines × ~120 chars = ~1.2k chars. Leaves ~3.8k for canonical sections under the overall 5k cap.

## Mutable-with-consolidation rule (shared across skills)

`overview.md` is mutable, not append-only. `craft:brainstorm` (at finish), `craft:implement` (at finish), and `craft:feature-state` (during compaction) may add, edit, merge, or remove entries when they spot duplicates, contradictions, or lines that fail the constraint-vs-description test. The discipline:

- **Check for duplicates before adding** (paraphrase match, not just exact-string match). One fact lives in exactly one place.
- **Resolve contradictions.** When a new decision overrides an old one, replace the old line; do not accumulate two contradicting entries.
- **Consolidate near-duplicates** that drifted into different wordings.
- **Atomize multi-fact lines** when you spot them. One fact per line.
- **Canonical entries are author-only:** skills propose, the user confirms.
- **Recent Changes index** is mechanical FIFO@10 - no consolidation needed there.

No `History` log. If you want to know when or why a line was added, the answer is `git log overview.md` (if the memory root is git-tracked) or the cited spec / changes file. The previous `History` section was an author-facing audit trail that drifted into bloat with no signal for a future Claude session.

This mirrors auto-memory's discipline: "update or remove memories that turn out to be wrong or outdated; do not write duplicates."

## Three buckets - the only canonical sections

1. **Product / Business** - why the feature exists, who it serves, what counts as done, explicit scope boundaries set by stakeholders. Recoverable from stakeholder conversations, not from code.
   - Example: *"Feature exists to satisfy the EU battery passport requirement - not a generic audit log."*
2. **Architectural Invariants** - constraints that must continue to hold; not descriptions of current code.
   - Example: *"Module A must never call Module B directly - all traffic goes through the event bus."*
   - Example: *"All new features in this domain must gate behind a feature-flag entry."*
3. **External Constraints** - compliance, legal, SLAs, contracts with other teams or systems.
   - Example: *"Response time must stay under 200ms; agreed with the mobile team for their cache TTL."*

If a line doesn't fit one of these three, it doesn't belong in canonical sections. Common offenders:

- "Guidelines" → fold real constraints into Architectural Invariants; drop the rest.
- "Core Decisions" describing how the code currently works → fail the constraint-vs-description test; drop or reword.

## Explicitly excluded (redirect to the right place)

- Function names, file paths, signatures, type shapes → read the code.
- Naming or refactor choices ("renamed X to Y") → git history.
- Implementation mechanics ("uses a dict keyed by user_id") → code.
- Per-session "what shipped" prose → `changes/YYYY-MM-DD-<topic>.md`.
- Session-level "we deferred X" → the relevant changes file's `Gaps left open` section.
- `Open Questions`, TODOs, `Recommended Next Steps` → ticket system or a non-loaded sibling file. `overview.md` is loaded every session; speculative items don't earn that cost.
- Rejected alternatives - unless the rationale is load-bearing for future decisions, in which case write it as a constraint: *"Do not adopt X - fails on Y."*
- Author-facing audit trail (dated `History` log of overview.md edits) → `git log overview.md` if the memory root is versioned.

## Template (only if creating overview.md fresh)

```markdown
# <Feature Name> - Overview

Long-lived constraints for <feature> plus a rolling index of recent sessions.

## Product / Business
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md

## Architectural Invariants
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md

## External Constraints
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md or external doc.

## Recent Changes
- YYYY-MM-DD: <short-topic> -> changes/YYYY-MM-DD-<short-topic>.md
```

## Writing to existing overview.md

- **Canonical sections:** put each new entry under the correct bucket. Apply the constraint-vs-description test before adding. Consolidate / replace stale or contradictory entries during the same edit when they're spotted. Do NOT mass-rewrite the file.
- **Recent Changes:** append at the top, trim to the last 10. Mechanical.
- **Bloat check:** when total file size approaches the 5k-char cap, prefer dropping stale canonical entries before resorting to manual line-trimming. The file decays into noise if it never sheds entries.
