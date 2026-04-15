# Dispatching a subagent (from craft:implement)

Default is main-session execution. Dispatching a subagent is the exception, not the rule, because the main session has context that a fresh agent does not and course-correction is cheap here but expensive there.

## When to dispatch

Only when **all** of these hold:

- The task is isolated and well-scoped (one or two files, no cross-cutting changes).
- The task does not change architecture, public APIs, or shared conventions.
- The work is mechanical enough that a fresh-context agent can do it correctly.

If any of the three is false, stay in the main session.

## How to dispatch

- Provide the full task text and the specific surrounding context. Never tell the subagent to "read the spec" or "figure it out".
- Require the subagent to ask clarifying questions before implementing if anything is ambiguous.
- Never dispatch two implementation subagents in parallel on overlapping files.
- Model selection: cheap model for mechanical single-file work, standard for multi-file integration, most capable for judgment calls.

## Handling the status report

Accept one of four statuses:

- `DONE` — proceed to review.
- `DONE_WITH_CONCERNS` — read the concerns; address correctness/scope issues before review, note observations and proceed.
- `NEEDS_CONTEXT` — supply the missing context and re-dispatch.
- `BLOCKED` — reassess: provide more context, use a more capable model, break the task smaller, or escalate.
