---
name: feature-state
description: Compact and consolidate a feature's `overview.md` so it stays small, current, and free of duplicates or contradictions. Use this whenever the user says "compact this feature", "the overview is getting noisy", "re-baseline the feature", "trim the changes index", "consolidate decisions", "feature-state", or whenever `overview.md` has drifted past its 5k-char cap or `Recent Changes` has more than 10 entries. On-demand only - other skills may prompt to invoke it but must not call it automatically. Does NOT write dated snapshots; the dated `snapshots/*.md` artifact is deprecated and read-only.
---

# Feature State (craft)

Compact `overview.md` for a feature so it stays small, current, and free of duplicates or contradictions. Auto-dream-shaped - the discipline is opportunistic compaction, not periodic re-distillation.

**Memory:** this skill reads and writes `<memory-root>/features/<slug>/overview.md` and reads `<memory-root>/features/<slug>/changes/*.md`. Path, layout, and invariants are defined in [`../../references/memory.md`](../../references/memory.md). File roles, read protocol, and write protocol live in [`../../references/feature-files.md`](../../references/feature-files.md). Format and consolidation rules for `overview.md` live in [`../../references/overview.md`](../../references/overview.md). Read all three before any memory access. `<memory-root>` refers to the path defined in `memory.md`.

**Announce at start:** "I'm using the craft:feature-state skill to compact <feature>."

**Deprecated behavior (do not do):** writing `snapshots/YYYY-MM-DD.md`, batched first-message protocol with the user, the canonical three-command branch/merge probe, "delta-only" snapshot rules. The dated snapshot artifact is gone. Existing `snapshots/*.md` on disk are legacy reference - never read by default, never edited, never deleted.

## What this skill does

1. Read `overview.md` and the changes files referenced by its `Recent Changes` index. Verify against current code (specs and old snapshots may be stale; code is truth).
2. **Consolidate canonical sections** (Product / Business, Architectural Invariants, External Constraints): drop entries the codebase no longer follows, merge near-duplicates, resolve contradictions.
3. **Promote from changes to canonical:** scan recent changes files for items that have hardened into invariants and propose adding them to canonical sections.
4. **Trim `Recent Changes` index** if it drifted past 10 entries (defense in depth - `craft:implement` should already do this).
5. **Flag bloat:** if `overview.md` still exceeds 5k chars after consolidation, propose specific entries to drop.

## Process

### 1. Confirm the feature in one message

Send a single short message to confirm the slug and the scope of work. Do not ask piecemeal.

```
I'm going to compact <slug>'s overview.md. Inputs to confirm:

1. Feature slug: <slug>  ← confirm or correct
2. overview.md size: <bytes> chars (cap 5k)
3. Recent Changes index: <N> entries (cap 10)
4. Changes files I'll review: <count from index>

I'll consolidate canonical sections, propose promotions from recent changes, trim the index, and flag any remaining bloat. No dated snapshot is written.

Reply with corrections or "go ahead".
```

Wait for the reply before reading heavy content.

### 2. Read code first, then memory

Code is the source of truth - changes files describe intent at write-time, code describes current reality.

- Read confirmed implementation files for the feature.
- Read `<memory-root>/features/<slug>/overview.md`.
- Read each changes file referenced by the `Recent Changes` index (load on demand; do not bulk-glob the directory). Older changes files (not in the index) are audit trail - skip unless investigating a specific past decision.
- Do NOT default-read legacy `snapshots/*.md`. Open one only if a current task explicitly references it as historical context.

### 3. Consolidate canonical sections in-place

Walk Product / Business, Architectural Invariants, External Constraints. For each entry:

- **Stale?** Codebase no longer follows it - propose removal (with `History` line noting why).
- **Contradicted?** A newer entry or recent code overrides it - propose replacement (with `History` line noting which old line was replaced).
- **Near-duplicate?** Two entries say the same thing in different words - propose merge.
- **Untouched?** Leave it.

Present the proposed changes as a single short diff list to the user. Apply on yes. Do not mass-rewrite without confirmation.

Format rule (caveman): each invariant stays one line, ≤ 200 chars, plus a Source citation. Multi-paragraph explanations belong in a changes file's `What shipped` section, not here.

### 4. Propose promotions from changes to canonical

Scan the changes files referenced in `Recent Changes`. Look for things that have hardened into invariants - decisions that now constrain future work, not just descriptions of one session's delta.

Strong promotion signals:
- The same constraint shows up in multiple recent changes files.
- A changes file's `What shipped` describes a new architectural rule ("from now on, X must Y") rather than a one-off implementation.
- The current code now relies on the decision in a way that future contributors will need to know about without reading a specific changes file.

Present candidates as a list with the proposed bucket. Apply on yes; add a `History` line citing the source changes file. Skip if there are no candidates.

### 5. Trim the Recent Changes index

If the index has more than 10 entries, trim to the most recent 10. The dropped entries' changes files remain on disk - they are simply no longer pointed at. This is defense in depth; `craft:implement` should already keep the index at 10.

### 6. Flag remaining bloat

After steps 3-5, check `overview.md` size:

- If ≤ 5k chars: done.
- If > 5k chars: identify the specific canonical entries most worth dropping (oldest, vaguest, or most narrowly scoped) and propose them. Apply on yes. Repeat until ≤ 5k or the user accepts the remaining size.

### 7. Hand off

Report:
- What was changed (added / merged / removed / promoted / trimmed counts).
- Final size of `overview.md` and `Recent Changes` length.
- If anything still feels off (e.g. a changes file that should probably be re-written), flag it but do not fix - that's the user's call.

## Red Flags

| Thought | Reality |
|---------|---------|
| "I'll write a new `snapshots/YYYY-MM-DD.md` to capture current state" | No. Snapshots are deprecated. The compacted `overview.md` is the new baseline. |
| "I'll bulk-read every changes file before consolidating" | No. Only the ones referenced by `Recent Changes`. Older changes are audit trail. |
| "I'll ask slug, then size, then scope - one at a time" | No. One batched first message. |
| "I'll mass-rewrite overview.md to make it tidier" | No. Propose specific consolidation changes; apply on confirmation. Each removal gets a `History` line. |
| "I'll edit a legacy `snapshots/*.md` to fix a contradiction" | No. Legacy snapshots are read-only. Fix `overview.md` instead. |
| "I'll edit an existing changes file to update a fact" | No. Changes files are immutable. Reflect the new fact in the next implement session's changes file or in `overview.md`'s canonical sections. |
| "Promotion is automatic - I'll move anything that looks like an invariant" | No. Each promotion goes to the user for explicit yes. |
| "The user just ran `craft:implement` so I should auto-run too" | No. This skill is invoked only on user request or when the loader trips a budget warning. |

## Integration

- Paired with `craft:implement`, which writes `changes/*.md` and updates the `Recent Changes` index at finish; never auto-invokes this skill.
- `craft:brainstorm` and `craft:implement` loaders may detect `overview.md` over 5k chars or `Recent Changes` over 10 entries and prompt the user once to invoke this skill - detection only, never auto-invoked.
