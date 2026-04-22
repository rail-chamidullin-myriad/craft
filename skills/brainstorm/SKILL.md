---
name: brainstorm
description: Brainstorm a feature, architecture change, or refactor and produce a dated design spec. Use this whenever the user says "brainstorm", "let's design X", "how should I build Y", "what's the best approach for Z", "should we use A or B", or is about to start any non-trivial feature, architecture change, or refactor — even if they don't explicitly say "brainstorm". Also use when deciding between two or more design directions. Collaborates via one clarifying question at a time (with a recommended answer), proposes 2-3 approaches with trade-offs, presents the design in sections for incremental approval, and writes a dated spec. Does NOT implement — handoff is to craft:implement in a separate session. Optional browser Visual Companion for UI mockups and visual A/B selection.
---

# Brainstorm (craft)

Turn an idea into a fully formed design through natural collaborative dialogue, then write a dated spec.

**Memory:** this skill reads and writes craft memory (specs, feature state, overview decisions). Path, layout, and invariants are defined once in the shared reference [`../../references/memory.md`](../../references/memory.md) — read it before any memory access. `<memory-root>` is used throughout this document to refer to the path defined there.

**Announce at start:** "I'm using the craft:brainstorm skill."

**Principles:** DRY. YAGNI. KISS. One question at a time. Recommend, don't lecture.

**Asking questions:** Ask when a decision genuinely needs the user's input - one question per turn, with your recommended answer. Skip anything you can verify from the code, `CLAUDE.md`, or repo conventions. Err on the side of fewer, higher-signal questions; brainstorming is a dialogue, not an interrogation.

<HARD-GATE>
Do NOT write code, scaffold, or invoke any implementation skill until you have presented a design and the user has approved it. The terminal state of this skill is a written spec plus a handoff note - not implementation.
</HARD-GATE>

## The Process

### 1. Load prior context (if feature exists)

Before asking any design questions:
- Look in `<memory-root>/features/<slug>/`.
- If `overview.md` exists, read it in full. These are canonical decisions, do not re-litigate unless the user flags them.
- If one or more `YYYY-MM-DD-<slug>-state.md` files exist, read the newest only. It is authoritative.
- Scan `<memory-root>/specs/` for prior specs on this topic. Do NOT read them by default, only reference them if the state file points at an unresolved question.

Restate what you loaded to the user before proceeding. Ask whether any of it is outdated.

### 2. Explore the codebase before asking

If a design question can be answered by reading the code, read the code. Don't ask the user things you can verify yourself. Check the project's `CLAUDE.md` and any project-specific convention files before asking anything that might already be a repo convention.

This discipline applies not only to clarifying questions to the user but also to items that would otherwise land in Open Questions. If a grep, file read, or command can resolve it, do that now - do not defer to the spec.

Before detailed questions, assess scope: if the request spans multiple independent subsystems, flag it and help decompose into sub-projects. Each sub-project gets its own spec later.

### 3. Ask clarifying questions

- One question per message. No bundled questions.
- Prefer multiple choice when natural, open-ended when not.
- Walk the decision tree depth-first: resolve a branch before moving to the next.
- Focus on *decisions*, not trivia.
- For each question, offer a recommended answer with a one-line rationale when you have one.

Probe briefly for the usual gaps when they seem load-bearing (error paths, empty input, partial failure, migration, auth, observability) but don't turn this into an interrogation. One focused question per gap, not a checklist.

### 4. Propose 2-3 approaches

Once the problem is clear, propose 2-3 approaches with trade-offs. Lead with your recommendation and explain why.

### 5. Present the design

Scale each section to its complexity: a few sentences if straightforward, up to 200-300 words if nuanced. Ask after each section whether it looks right. Cover: architecture, components, data flow, error handling, testing. Go back and clarify when something doesn't fit.

**Design for isolation and clarity:** smaller, well-bounded units with clear interfaces. For each unit you should be able to answer what it does, how to use it, and what it depends on.

**In existing codebases:** follow existing patterns. Include targeted cleanup only where it serves the current goal, no unrelated refactoring.

### 6. Use the Visual Companion when the answer is visual

If upcoming questions involve visual content (mockups, layouts, diagrams), offer the browser-based companion once as its own message — do not combine with other content:

> "Some of what we're working on might be easier to explain if I can show it to you in a web browser. I can put together mockups, diagrams, comparisons, and other visuals as we go. This feature is still new and can be token-intensive. Want to try it? (Requires opening a local URL)"

If accepted, read `visual-companion.md` in this skill directory for the full guide (server lifecycle, HTML fragments, reading selections, mockup persistence). Even after acceptance, decide per question: browser only when the user would understand it better by *seeing* than *reading*. A question about a UI topic is not automatically a visual question — "what kind of wizard do you want?" is conceptual (terminal); "which of these wizard layouts feels right?" is visual (browser).

### 7. Self-check before writing

Before writing the spec, scan for these issues (fix inline, no need to re-review):

| Category | What to catch |
|----------|--------------|
| Completeness | `TBD`, `TODO`, "figure out later", or sections left blank |
| Consistency | Decisions that contradict each other, or contradict `overview.md` |
| Clarity | Requirements ambiguous enough that two readers would build different things |
| Scope | Spec covers multiple independent subsystems - if so, split before writing |
| YAGNI | Features the user didn't ask for, or abstractions with no current caller |
| Simplicity | Would a senior engineer call this overcomplicated? If yes, cut until they wouldn't. |
| Open Questions discipline | For each entry, ask "can I answer this by reading code or running a grep?" If yes, resolve inline and delete the entry. This section is not a deferral pile. |

### 8. Write the spec

Ensure the target directory exists (`mkdir -p <memory-root>/specs/`). If you used the Visual Companion this session, also ensure `.craft/` is in `.gitignore` - mockups persist under `<project>/.craft/brainstorm/` and should not be tracked.

Write to `<memory-root>/specs/YYYY-MM-DD-<topic>-design.md` using the template in `templates/spec.md`.

### 9. Confirm before handoff

Do not ask the user to open the file. The spec typically lives outside the working tree (under the configured memory root) and is awkward to browse; more importantly, the user already saw and approved each section during step 5 — the written spec is the record, not a new deliverable for them to review.

After writing, send a compact recap of the key decisions (3-5 bullets, drawn from what was already agreed) and ask:

> "Spec written to `<path>`. Recap above — any corrections before we move to implementation?"

If they request changes, apply them, re-run the self-check, and send a fresh recap. Proceed once they approve.

### 10. Offer feature-state update and hand off

Ask:

> Do you want to update the feature state snapshot? This distills current state so the next brainstorming session can start from one file instead of the spec pile.

If yes, invoke `craft:feature-state`. If no, skip.

Also ask whether any decision from this session is canonical enough to append to `overview.md`. Only append on explicit yes.

Then report:
- Path to the new spec file.
- Any open questions still to resolve.
- Suggest: the implementation session can read this spec and invoke `craft:implement`.

## Red Flags

| Thought | Reality |
|---------|---------|
| "Let me propose the full design in one message" | No. Sections, with approval between them. |
| "I will ask all 12 questions at once" | No. Depth-first, one per turn. |
| "I will skip reading the state file since I just had a session on this" | No. Read it. A new session has no memory. |
| "I will start implementing because the user said yes once" | No. This skill ends at the spec. Implementation is a separate session via craft:implement. |
| "I will open the browser for every question" | No. Visual only when the answer is visual. |
| "The spec is a summary of what we discussed" | No. The spec captures decisions + rationale + open questions, not a chat transcript. |
| "I'll note this as an open question and let implementation resolve it" | No. If a grep or file read answers it, run the grep or read the file now. Open Questions is for items only the user can decide or that genuinely need implementation-time context. |

## Integration

- Pairs with `craft:implement` (next session reads the spec and implements it).
- Pairs with `craft:feature-state` (optional finish step to update the state snapshot).
