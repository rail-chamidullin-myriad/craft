# Compaction Audit

Keep `overview.md` small: every loaded line is a current constraint on future code, phrased telegraphically. No descriptions of current code, no duplicates, no audit trail, no TODOs.

Run by `craft:brainstorm` and `craft:implement` at finish, and on explicit user request ("compact this feature"). There is no standalone compaction skill; this reference is the whole mechanism.

## When to run

At a writing skill's finish step, run the audit when any holds:
- `overview.md` > 4k chars (approaching the 5k cap).
- `Recent Changes` index > 10 entries.
- Visible drift: duplicates, contradictions, or description-lines spotted this session.

Otherwise skip. A file born filtered + terse (per the write protocol in `feature-files.md`) rarely trips these.

On explicit user request, run regardless of size.

## Two passes

Prerequisite: read `overview.md`; read only the `changes/*.md` referenced by the `Recent Changes` index (on demand, no bulk glob); read confirmed implementation files (code is the source of truth). Do NOT default-read legacy `snapshots/*.md`.

### Pass 1 - semantic (cut)

Walk each line under Product / Business, Architectural Invariants, External Constraints:

1. **Constraint or description?** Stress test (`overview.md`): "if the implementation flipped tomorrow - different storage, shape, module - would this line need rewriting?" Yes -> description; drop or reword to the underlying constraint. Biggest source of bloat.
2. **Atomic?** One fact per line. Split or trim bundled lines.
3. **<= 200 chars incl. citation?** Long lines usually fail check 1 or 2.
4. **Unique?** Paraphrase-match, not exact. One home per fact.
5. **Citation healthy?** A single pointer (`Source: specs/<file>.md` or `Source: session YYYY-MM-DD`), not provenance prose.
6. **Stale against code?** Removed constraint -> drop; contradicted -> replace.

Then:
- **Drop deprecated sections wholesale:** `Open Questions`, `Recommended Next Steps`, `History`, `Guidelines` (fold real constraints into Architectural Invariants first). Rename/fold any other non-canonical section into the three buckets.
- **Trim `Recent Changes`** to the last 10 (defense in depth).
- **Hub vs leaf:** a hub `overview.md` is reference-dominant (mostly `## Related Features`, few/no constraints); a leaf stays constraint-rich. Do not strip a leaf's constraints down to links.

### Pass 2 - stylistic (compress)

Rewrite surviving prose into telegraphic form per `writing-style.md`. This is where density comes from once junk is gone.

## Author-gating

- **Substance changes** (add / remove / change a constraint's meaning, drop a section) are proposed to the user as one short diff list; apply on yes. Do not mass-rewrite unannounced.
- **Meaning-preserving stylistic rewrites** (Pass 2) are mechanical; apply without re-confirmation.

## After

Report counts (audited / dropped / merged / sections removed / index trimmed) and final `overview.md` size. If a `changes/*.md` looks like it should be rewritten, flag it - do not edit it (changes files are immutable).
