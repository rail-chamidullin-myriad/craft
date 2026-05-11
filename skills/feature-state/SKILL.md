---
name: feature-state
description: Compact and consolidate a feature's `overview.md` so it stays small, current, and free of duplicates or contradictions. Use this whenever the user says "compact this feature", "the overview is getting noisy", "re-baseline the feature", "trim the changes index", "consolidate decisions", "feature-state", or whenever `overview.md` has drifted past its 5k-char cap or `Recent Changes` has more than 10 entries. On-demand only - other skills may prompt to invoke it but must not call it automatically. Does NOT write dated snapshots; the dated `snapshots/*.md` artifact is deprecated and read-only.
---

# Feature State (craft)

Compact `overview.md` for a feature so every loaded line is a current constraint on future code - no descriptions of current code, no duplicates, no audit trail, no TODOs.

**Memory:** this skill reads and writes `<memory-root>/features/<slug>/overview.md` and reads `<memory-root>/features/<slug>/changes/*.md`. Path, layout, and invariants are defined in [`../../references/memory.md`](../../references/memory.md). File roles, read protocol, and write protocol live in [`../../references/feature-files.md`](../../references/feature-files.md). Format, filter, constraint-vs-description test, and consolidation rules for `overview.md` live in [`../../references/overview.md`](../../references/overview.md). Read all three before any memory access. `<memory-root>` refers to the path defined in `memory.md`.

**Announce at start:** "I'm using the craft:feature-state skill to compact <feature>."

**Deprecated behavior (do not do):** writing `snapshots/YYYY-MM-DD.md`, batched first-message protocol with the user, the canonical three-command branch/merge probe, "delta-only" snapshot rules, dated `History` line bookkeeping. The dated snapshot artifact is gone. Existing `snapshots/*.md` on disk are legacy reference - never read by default, never edited, never deleted. The `History` section inside `overview.md` is also gone - if you see one in an existing overview, propose removing it wholesale.

## What this skill does

1. **Per-line audit of canonical sections.** For each entry under Product / Business, Architectural Invariants, and External Constraints, ask: constraint or description? atomic? unique? maintainable citation? Propose corrections.
2. **Drop deprecated sections.** Propose removing any `Open Questions`, `Recommended Next Steps`, `History`, or `Guidelines` sections wholesale. Fold genuine constraints buried in `Guidelines` into `Architectural Invariants` before dropping.
3. **Promote from changes to canonical.** Scan recent changes files for items that have hardened into constraints and propose adding them.
4. **Trim `Recent Changes` index** if it drifted past 10 entries (defense in depth - `craft:implement` should already do this).
5. **Flag bloat.** If `overview.md` still exceeds 5k chars after the above, propose specific entries to drop.

## Process

### 1. Confirm the feature in one message

Send a single short message to confirm the slug and the scope of work. Do not ask piecemeal.

```
I'm going to compact <slug>'s overview.md. Inputs to confirm:

1. Feature slug: <slug>  ← confirm or correct
2. overview.md size: <bytes> chars (cap 5k)
3. Recent Changes index: <N> entries (cap 10)
4. Changes files I'll review: <count from index>
5. Deprecated sections present: <Open Questions | Recommended Next Steps | History | Guidelines | none>

I'll: audit each canonical entry per line, propose dropping deprecated sections, propose promotions from recent changes, trim the index, and flag any remaining bloat. No dated snapshot is written.

Reply with corrections or "go ahead".
```

Wait for the reply before reading heavy content.

### 2. Read code first, then memory

Code is the source of truth - changes files describe intent at write-time, code describes current reality.

- Read confirmed implementation files for the feature.
- Read `<memory-root>/features/<slug>/overview.md`.
- Read each changes file referenced by the `Recent Changes` index (load on demand; do not bulk-glob the directory). Older changes files (not in the index) are audit trail - skip unless investigating a specific past decision.
- Do NOT default-read legacy `snapshots/*.md`. Open one only if a current task explicitly references it as historical context.

### 3. Per-line audit of canonical sections

Walk each line under Product / Business, Architectural Invariants, and External Constraints. For each, run these checks in order:

1. **Constraint or description?** Apply the stress test from `references/overview.md`: *"if the implementation flipped tomorrow - different storage, different shape, different module - would this line need rewriting?"* If yes, it's a description - propose dropping or rewording to the underlying constraint. This is the biggest source of stale memory.
2. **Atomic?** Single-fact lines only. If the entry bundles multiple facts in one sentence (rule + anecdote + PR number + caveat), propose splitting into separate lines or trimming to just the rule.
3. **≤ 200 chars including citation?** Long lines almost always fail the description check or the atomic check - tighten or split.
4. **Unique?** If a paraphrase of this fact appears elsewhere in `overview.md` (often in `Feature Boundaries` or `Guidelines`), propose keeping one home and removing the other.
5. **Citation healthy?** Bloated provenance prose ("Source: session 2026-04-14, PR #4836 review from Jan Plesnik; commit `d37ebdba4`") gets trimmed to a single pointer (`Source: specs/<file>.md` or `Source: session YYYY-MM-DD`). Citations are pointers, not authority - code is the source of truth.
6. **Stale against current code?** A constraint the codebase no longer follows: propose removal. A constraint contradicted by a newer entry or recent code: propose replacement.

Present all proposed changes as one short diff list to the user. Apply on yes. Do NOT mass-rewrite without confirmation. No `History` line bookkeeping - the section is gone.

### 4. Drop deprecated sections wholesale

If `overview.md` contains any of these sections, propose removing them entirely:

- `Open Questions` - speculative TODOs Claude can't act on, never shed entries. Belong in a ticket system, not always-loaded memory.
- `Recommended Next Steps` - same problem in a different hat.
- `History` - author-facing audit trail of overview edits. Recoverable from `git log overview.md` (if memory root is versioned) or the cited spec / changes files.
- `Guidelines` - not one of the three canonical buckets. Fold real constraints into `Architectural Invariants` (after applying the constraint-vs-description test to each); drop the rest.

Also propose removing or normalizing any other non-canonical section name (`Core Decisions`, `Feature Boundaries`, custom names) - either rename and fold into the three canonical buckets, or drop entries that don't fit any bucket.

Present as a single diff: "removing these N sections; folding these M entries from `Guidelines` into `Architectural Invariants` after rewording as constraints." Apply on yes.

### 5. Propose promotions from changes to canonical

Scan the changes files referenced in `Recent Changes`. Look for things that have hardened into constraints - decisions that now bind future work, not just descriptions of one session's delta.

Strong promotion signals:
- The same constraint shows up in multiple recent changes files.
- A changes file's `What shipped` describes a new architectural rule ("from now on, X must Y") rather than a one-off implementation.
- The current code now relies on the decision in a way that future contributors will need to know without reading a specific changes file.

Each promoted line must itself pass the constraint-vs-description test before adding. Present candidates as a list with the proposed bucket. Apply on yes. Skip if there are no candidates.

### 6. Trim the Recent Changes index

If the index has more than 10 entries, trim to the most recent 10. The dropped entries' changes files remain on disk - they are simply no longer pointed at. This is defense in depth; `craft:implement` should already keep the index at 10.

### 7. Flag remaining bloat

After steps 3-6, check `overview.md` size:

- If ≤ 5k chars: done.
- If > 5k chars: identify the specific canonical entries most worth dropping (oldest, vaguest, or most narrowly scoped) and propose them. Apply on yes. Repeat until ≤ 5k or the user accepts the remaining size.

### 8. Hand off

Report:
- What was changed (audited / merged / removed / promoted / sections dropped / index trimmed - counts).
- Final size of `overview.md` and `Recent Changes` length.
- If anything still feels off (e.g. a changes file that should probably be re-written), flag it but do not fix - that's the user's call.

## Red Flags

| Thought | Reality |
|---------|---------|
| "This entry describes how the code currently works - that's useful context" | No. If a future code change would require rewriting the line, it's a description, not a constraint. Reword or drop. |
| "I'll keep the `History` section for evolution context" | No. The section is gone. Provenance lives in `git log overview.md` and the cited spec / changes files. Per-session evolution lives in `changes/*.md`. |
| "Open Questions is small, leave it" | No. Drop wholesale. It only ever grows; entries don't earn the always-loaded cost. |
| "I'll write a new `snapshots/YYYY-MM-DD.md` to capture current state" | No. Snapshots are deprecated. The compacted `overview.md` is the new baseline. |
| "I'll bulk-read every changes file before consolidating" | No. Only the ones referenced by `Recent Changes`. Older changes are audit trail. |
| "I'll ask slug, then size, then scope - one at a time" | No. One batched first message. |
| "I'll mass-rewrite overview.md to make it tidier" | No. Propose specific consolidation changes per line; apply on confirmation. |
| "I'll edit a legacy `snapshots/*.md` to fix a contradiction" | No. Legacy snapshots are read-only. Fix `overview.md` instead. |
| "I'll edit an existing changes file to update a fact" | No. Changes files are immutable. Reflect the new fact in the next implement session's changes file or in `overview.md`'s canonical sections. |
| "Promotion is automatic - I'll move anything that looks like an invariant" | No. Each promotion goes to the user for explicit yes, and must pass the constraint-vs-description test first. |
| "The user just ran `craft:implement` so I should auto-run too" | No. This skill is invoked only on user request or when the loader trips a budget warning. |

## Integration

- Paired with `craft:implement`, which writes `changes/*.md` and updates the `Recent Changes` index at finish; never auto-invokes this skill.
- `craft:brainstorm` and `craft:implement` loaders may detect `overview.md` over 5k chars or `Recent Changes` over 10 entries and prompt the user once to invoke this skill - detection only, never auto-invoked.
