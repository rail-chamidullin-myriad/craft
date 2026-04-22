# <Topic> - Design (YYYY-MM-DD)

**Feature slug:** <slug> (if part of a larger feature)
**Status:** design
**Replaces:** specs/YYYY-MM-DD-<topic>-design.md (if superseding an earlier spec)

## Context
Why are we doing this? What problem does it solve? What is the current state?

## Goal
One paragraph. What does "done" look like?

## Success Criteria
Verifiable checks that prove the design was implemented correctly. Each item should be something an implementer (or test) can confirm - not vague aspirations. Examples: "endpoint returns 404 for unknown IDs", "page loads in under 200ms on dev hardware", "feature flag X gates the new path". Implementation verifies against this list.

## Design
Architecture, key components, interfaces, data flow. Include diagrams if produced via Visual Companion (link or embed screenshots).

## Decisions Made
- Chose A over B because ...
- Chose C because ...

## Open Questions
- Items only the user can decide, or items that genuinely need implementation-time context (you will know more after writing code). Do NOT use this section for anything a grep, file read, or command could answer now - resolve those before writing the spec. If every entry reads "grep for X" or "check if Y exists", you shipped the spec too early. If nothing genuinely open remains, write "None." and move on.

## Out of Scope
- What this spec deliberately does NOT cover.
