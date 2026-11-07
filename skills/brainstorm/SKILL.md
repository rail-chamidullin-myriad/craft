---
name: brainstorm
description: Design companion for a feature, architecture change, or refactor. Briefs the user on how the area works today, what the request changes, and which decisions and gaps are open, then helps them decide through discussion, audits, comparisons, and mockups. Writes a dated design spec once the decisions settle. Use when the user says "brainstorm", "let's design X", "how should I build Y", "what's the best approach for Z", "should we use A or B", or starts a non-trivial feature, architecture change, or refactor.
---

# Brainstorm (craft)

Help the user reach a design they trust. Brief them first, then follow their lead, and record the result as a spec.

Specs and feature overviews live in craft memory. Read [`../../references/memory.md`](../../references/memory.md) before you touch it.

## 1. Gather context

- Read the feature's `overview.md` when one exists. Say what you loaded and ask whether any of it is outdated.
- Read the code the request touches, the project's `CLAUDE.md`, and its conventions. Settle anything the code can answer before you ask.
- If the request spans independent subsystems, propose splitting it.

## 2. Brief the user

Open with a briefing, before any question:

1. **Today:** how the area works now, with a short flow sketch.
2. **The change:** what the request changes, in two or three sentences.
3. **Decisions:** each open decision with its realistic options and your recommendation.
4. **Gaps and risks:** edge cases, missing data, and conflicts with existing rules or `overview.md`.

Example:

```
Today: reports expire after 30 days. A nightly job deletes them.
  save -> storage (30-day expiry) -> nightly purge
The change: paid users keep reports until they delete them.
Decisions:
  1. Where retention lives: per plan in billing (recommended, one rule) or a flag per report.
  2. On downgrade: keep existing reports (recommended) or purge after a grace period.
Gaps: no test covers the purge job. Two other services read the expiry date.
```

## 3. Decide together

Follow the user's lead. Pick whichever move helps the decision at hand:

- **Discuss:** answer, go deeper on one decision, or ask questions. Ask independent questions together, each with a recommended answer.
- **Audit:** when evidence beats opinion, check call sites, current behavior on real data, usage numbers, or how sibling code solves the same problem.
- **Compare:** put two approaches side by side, or challenge an assumption, the user's included.
- **Show:** when seeing beats reading, publish a mockup or diagram as an Artifact. Load the `artifact-design` skill first, and revise the same file so the URL stays.

Keep the design proportional to the problem. Follow existing patterns, and build only what the request needs.

## 4. Write the spec

When the main decisions settle, offer to write the spec. Before you write it, check it for `TBD`s, contradictions with `overview.md`, open questions the code can answer, and parts nobody asked for.

Write `<memory-root>/specs/YYYY-MM-DD-<topic>-design.md` from [`../../references/templates/spec.md`](../../references/templates/spec.md). A spec stays as written. A changed design gets a new spec that names the one it replaces.

Reply with the path and a recap of the key decisions in three to five bullets.

## 5. Update feature memory

1. Create the feature's `overview.md` if it is missing.
2. Add the spec to its Recent Changes list.
3. Propose constraint lines for decisions that outlive this change. Apply them when the user agrees.
4. Compact `overview.md` if it is over budget.

## 6. Build it

When you build the design in this session, check each success criterion in the spec, and propose an `overview.md` update if a lasting decision changed.
