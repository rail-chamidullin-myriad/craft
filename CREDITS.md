# Credits

Craft is a thin opinionation layer over ideas pioneered by the **superpowers** plugin by Jesse Vincent: https://github.com/obra/superpowers

## Vendored code

The Visual Companion infrastructure in `skills/brainstorm/scripts/` is copied from `superpowers:brainstorming`:

- `frame-template.html`
- `helper.js`
- `server.cjs`
- `start-server.sh`
- `stop-server.sh`

Plus the accompanying `skills/brainstorm/visual-companion.md` guide.

superpowers is MIT-licensed. See https://github.com/obra/superpowers/blob/main/LICENSE for the original license text.

## Adapted ideas

The following disciplines in `skills/brainstorm/SKILL.md`, `skills/implement/SKILL.md`, and `skills/feature-state/SKILL.md` are adapted (not copied verbatim) from superpowers skills:

- **`superpowers:brainstorming`** - checklist-driven flow (explore context → clarifying questions one at a time → propose 2-3 approaches → present design in sections → self-check → user review), Visual Companion usage guidance, hard-gate before implementation.
- **`superpowers:writing-plans`** - spec self-review checklist (Completeness, Consistency, Clarity, Scope, YAGNI).
- **`superpowers:executing-plans` / `superpowers:subagent-driven-development`** - critical-review-before-executing, stop-on-blocker discipline, subagent status codes.
- **`superpowers:test-driven-development`** - TDD cadence (failing test → run → impl → run → commit).

Craft deliberately drops the plan-on-disk and per-task-subagent orchestration in favor of spec + TodoWrite + main-session-loop. The shape of the workflow differs; the underlying disciplines are superpowers'.
