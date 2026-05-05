# overview.md guidance

`overview.md` is the always-loaded anchor for a feature. It is small, mutable, and has two kinds of content:

1. **Canonical sections** - long-lived decisions (product/business, architectural invariants, external constraints) that outlive any single implementation. Author-only, evolved with consolidation discipline.
2. **Recent Changes index** - one line per recent memorable session (brainstorm or implement), newest top, FIFO trimmed to the last 10. Mechanically managed by `craft:brainstorm` (pointing at a spec file) and `craft:implement` (pointing at a changes file).

Plus optional `Open Questions` and a `History` log of dated additions to the canonical sections.

**Hard cap: 5k chars (~1.2k tokens).** This file is loaded by every craft session on the feature. Bloat taxes every future load.

## The filter (one line)

If a future reader could answer it by reading the code, it does NOT belong in the canonical sections of `overview.md`.

## Canonical-sections format rule (caveman)

**Each invariant is ONE LINE, ≤ 200 chars, plus a Source citation on the same line or next line.** No multi-paragraph explanations, no "do not do X" warnings, no rationale prose. The invariant statement should be self-evidently load-bearing; if it needs explaining, it belongs in a changes file's `What shipped` section, not here.

The `History` section: one line per dated session, comma-separated bullet of what was added.

## Recent Changes section spec

- Newest entry at the top. FIFO trim to the last 10.
- One line per entry. Format depends on the writing skill:
  - `craft:brainstorm` at finish: `- YYYY-MM-DD: <short-topic> -> specs/YYYY-MM-DD-<short-topic>-design.md`
  - `craft:implement` at finish: `- YYYY-MM-DD: <short-topic> -> changes/YYYY-MM-DD-<short-topic>.md`
- Both skills append mechanically: at the top, trim to 10. No judgment.
- Spec files live under `<memory-root>/specs/`; changes files live under `<memory-root>/features/<slug>/changes/`. Both load on demand only.
- Older entries (past the 10th most recent) stay on disk as audit trail; they are simply not pointed at from this index.
- Index budget: 10 lines × ~120 chars = ~1.2k chars. Leaves ~3.8k for canonical sections + history under the overall 5k cap.

## Mutable-with-consolidation rule (shared across skills)

`overview.md` is mutable, not append-only. `craft:brainstorm` (at finish), `craft:implement` (at finish), and `craft:feature-state` (during compaction) may add, edit, merge, or remove entries when they spot duplicates or contradictions. The discipline:

- **Check for duplicates before adding** (paraphrase match, not just exact-string match).
- **Resolve contradictions.** When a new decision overrides an old one, replace the old line; do not accumulate two contradicting entries.
- **Consolidate near-duplicates** that drifted into different wordings.
- **Canonical entries are author-only**: skills propose, the user confirms. Adds get a dated `History` line. Removals during consolidation also get a `History` line noting why.
- **Recent Changes index** is mechanical FIFO@10 - no consolidation needed there.

This mirrors auto-memory's discipline: "update or remove memories that turn out to be wrong or outdated; do not write duplicates."

## Three buckets — only log canonical entries that fit one of these

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
- Per-session "what shipped" prose → `changes/YYYY-MM-DD-<topic>.md`.
- Session-level "we deferred X" → that's what the relevant changes file's `Gaps left open` section is for.
- Rejected alternatives, unless the rationale is load-bearing for future decisions.

## Template (only if creating overview.md fresh)

```markdown
# <Feature Name> - Overview

Long-lived decisions for <feature> plus a rolling index of recent implement sessions.

## Product / Business
- <Decision>. Source: specs/YYYY-MM-DD-<topic>-design.md

## Architectural Invariants
- <Invariant>. Source: specs/YYYY-MM-DD-<topic>-design.md

## External Constraints
- <Constraint>. Source: specs/YYYY-MM-DD-<topic>-design.md or external doc.

## Recent Changes
- YYYY-MM-DD: <short-topic> -> changes/YYYY-MM-DD-<short-topic>.md

## Open Questions
- <Question>.

## History
- YYYY-MM-DD: added <decision> under <bucket> (source: spec or session).
```

## Writing to existing overview.md

- **Canonical sections:** put each new entry under the correct bucket and add a `History` line. Consolidate / replace stale or contradictory entries during the same edit when they're spotted; add a `History` line noting the removal too. Do NOT mass-rewrite the file.
- **Recent Changes:** append at the top, trim to the last 10. Mechanical.
- **Bloat check:** when total file size approaches the 5k-char cap, prefer dropping stale canonical entries before resorting to manual line-trimming. The file decays into noise if it never sheds entries.
