---
name: brainstorm
description: Brainstorm a feature or update and produce a dated design spec. Collaborates with the user through clarifying questions (one at a time), proposes 2-3 approaches, and presents a design for approval before writing the spec. Uses an optional browser-based Visual Companion for UI mockups, diagrams, and visual A/B selection. Writes the spec to docs/craft/specs/.
---

# Brainstorm (craft)

Turn an idea into a fully formed design through natural collaborative dialogue, then write a dated spec. Adapted from `superpowers:brainstorming` with craft-specific paths and handoffs.

**Default artifact root:** `docs/craft/` at the repo root. A project's `CLAUDE.md` or `CLAUDE.local.md` may override this (e.g. a personal subdirectory). If overridden, use the override path everywhere this skill writes or reads `docs/craft/`.

**Announce at start:** "I'm using the craft:brainstorm skill."

**Principles:** DRY. YAGNI. KISS. One question at a time. Recommend, don't lecture.

## When to use

- Before any non-trivial feature, architecture change, or refactor.
- When you need to decide between two or more design directions.
- Triggers: "brainstorm", "let's design X", "help me figure out how to build Y".

Do NOT jump to implementation from this skill. Implementation happens in a separate session via `craft:implement`.

<HARD-GATE>
Do NOT write code, scaffold, or invoke any implementation skill until you have presented a design and the user has approved it. The terminal state of this skill is a written spec plus a handoff note - not implementation.
</HARD-GATE>

## Checklist

Complete these in order:

1. **Load prior context** - feature state, overview, existing specs (if this is continued work).
2. **Explore project context** - relevant files, repo conventions, recent commits.
3. **Offer Visual Companion** (if upcoming questions look visual) - its own message, no other content.
4. **Ask clarifying questions** - one at a time, understand purpose, constraints, success criteria.
5. **Propose 2-3 approaches** - with trade-offs and your recommendation.
6. **Present the design** - in sections scaled to complexity, get approval after each section.
7. **Self-check the design** - quick inline pass for placeholders, contradictions, ambiguity, scope.
8. **Write the spec** - save to `docs/craft/specs/YYYY-MM-DD-<topic>-design.md`.
9. **User reviews the written spec** - ask before handing off.
10. **Hand off to craft:implement** - suggest implementation in a fresh session.

## The Process

### 1. Load prior context (if feature exists)

Before asking any design questions:
- Look in `docs/craft/features/<slug>/`.
- If `overview.md` exists, read it in full. These are canonical decisions, do not re-litigate unless the user flags them.
- If one or more `YYYY-MM-DD-<slug>-state.md` files exist, read the newest only. It is authoritative.
- Scan `docs/craft/specs/` for prior specs on this topic. Do NOT read them by default, only reference them if the state file points at an unresolved question.

Restate what you loaded to the user before proceeding. Ask whether any of it is outdated.

### 2. Explore the codebase before asking

If a design question can be answered by reading the code, read the code. Don't ask the user things you can verify yourself. Check `.claude/rules/` and `CLAUDE.md` before asking anything that might already be a repo convention.

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

Per-question decision: would the user understand this better by *seeing* it than *reading* it?

Use browser for:
- UI mockups, wireframes, layouts, component shapes.
- Architecture diagrams, data flow, state machines as diagrams.
- Side-by-side visual comparisons.

Use terminal for:
- Requirements, scope, conceptual A/B/C picks between described approaches.
- Tradeoff lists, pros/cons tables.
- API design, data modeling, naming.

A question *about* a UI topic isn't automatically a visual question. "What kind of wizard do you want?" is conceptual (terminal). "Which of these wizard layouts feels right?" is visual (browser).

See `visual-companion.md` in this skill directory for the full guide.

**Starting the server:**

```bash
# From the plugin dir:
scripts/start-server.sh --project-dir /path/to/project
```

Save `screen_dir` and `state_dir` from the JSON response. Write HTML fragments to `screen_dir`; read user selections from `state_dir/events` on the next turn.

Mockups persist under `<project>/.superpowers/brainstorm/`. Ensure `.superpowers/` is in `.gitignore`.

**Stopping the server:** `scripts/stop-server.sh` at end of session.

### 7. Self-check before writing

Before writing the spec, scan for these issues (fix inline, no need to re-review):

| Category | What to catch |
|----------|--------------|
| Completeness | `TBD`, `TODO`, "figure out later", or sections left blank |
| Consistency | Decisions that contradict each other, or contradict `overview.md` |
| Clarity | Requirements ambiguous enough that two readers would build different things |
| Scope | Spec covers multiple independent subsystems - if so, split before writing |
| YAGNI | Features the user didn't ask for, or abstractions with no current caller |

### 8. Write the spec

Write to `docs/craft/specs/YYYY-MM-DD-<topic>-design.md`:

```markdown
# <Topic> - Design (YYYY-MM-DD)

**Feature slug:** <slug> (if part of a larger feature)
**Status:** design
**Replaces:** specs/YYYY-MM-DD-<topic>-design.md (if superseding an earlier spec)

## Context
Why are we doing this? What problem does it solve? What is the current state?

## Goal
One paragraph. What does "done" look like?

## Design
Architecture, key components, interfaces, data flow. Include diagrams if produced via Visual Companion (link or embed screenshots).

## Decisions Made
- Chose A over B because ...
- Chose C because ...

## Open Questions
- Anything not resolved in this session that a later brainstorm or implementation must address.

## Out of Scope
- What this spec deliberately does NOT cover.
```

### 9. User reviews the written spec

After writing, ask:

> "Spec written to `<path>`. Please review it and let me know if you want any changes before we move to implementation."

Wait. If they request changes, apply them and re-run the self-check. Only proceed once the user approves.

### 10. Offer feature-state update and hand off

Ask:

> Do you want to update the feature state snapshot? This distills current state so the next brainstorming session can start from one file instead of the spec pile.

If yes, invoke `craft:feature-state`. If no, skip.

Also ask whether any decision from this session is canonical enough to append to `overview.md`. Only append on explicit yes.

Then report:
- Path to the new spec file.
- Any open questions still to resolve.
- Suggest: the implementation session can read this spec and invoke `craft:implement`.

## Key Principles

- **One question at a time** - don't overwhelm.
- **Multiple choice preferred** when natural.
- **YAGNI ruthlessly** - strip unnecessary features.
- **Explore alternatives** - propose 2-3 approaches before settling.
- **Incremental validation** - present the design in sections, confirm each.
- **Be flexible** - go back and clarify when something doesn't fit.

## Red Flags

| Thought | Reality |
|---------|---------|
| "Let me propose the full design in one message" | No. Sections, with approval between them. |
| "I will ask all 12 questions at once" | No. Depth-first, one per turn. |
| "I will skip reading the state file since I just had a session on this" | No. Read it. A new session has no memory. |
| "I will start implementing because the user said yes once" | No. This skill ends at the spec. Implementation is a separate session via craft:implement. |
| "I will open the browser for every question" | No. Visual only when the answer is visual. |
| "The spec is a summary of what we discussed" | No. The spec captures decisions + rationale + open questions, not a chat transcript. |

## Upstream Sync

This skill adapts `superpowers:brainstorming`. Scripts in `scripts/` are vendored from the upstream skill. To check for upstream improvements:

```bash
diff -r ~/.claude/plugins/cache/claude-plugins-official/superpowers/<latest>/skills/brainstorming/ .claude/plugins/craft/skills/brainstorm/
```

Read the diff; port selectively. Craft-specific conventions (spec path override, state-file preload, handoff to craft:implement instead of writing-plans) must be preserved.

## Integration

- Pairs with `craft:implement` (next session reads the spec and implements it).
- Pairs with `craft:feature-state` (optional finish step to update the state snapshot).
