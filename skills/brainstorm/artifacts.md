# Visuals via Artifacts

Publish mockups, diagrams, and visual A/B comparisons as Claude Code Artifacts — self-contained web pages hosted on `claude.ai`. The user opens a link, looks, and responds in the terminal.

## When to Use

You render a visual, the user views it in the browser, the user replies in the terminal — that reply is the feedback.

Use it **only when the content is a rendered visual** — and only then:

- UI mockups — wireframes, layouts, navigation structures, component designs
- Architecture diagrams — components, data flow, relationship maps, state machines
- Side-by-side visual comparisons — two layouts, two color schemes, two design directions
- Design polish — look and feel, spacing, visual hierarchy

Do **not** use it for words. Requirements, scope, tradeoff lists, approach A/B/C described in prose, API/data-model decisions, and any clarifying question all belong in the terminal (plain text or `AskUserQuestion`). An Artifact whose body is a list of text options is strictly worse than asking directly — it adds a round-trip and a browser tab for nothing.

The test for every question: would the user understand this better by **seeing** it than **reading** it? A question *about* a UI topic is not automatically visual.

## Requirements

Artifacts need a Team/Enterprise plan, a `claude.ai` login (`/login`, not an API key or cloud-provider credential), and the Anthropic API provider. They are off by default in SDK / GitHub Actions / MCP contexts. If publishing is unavailable, describe the option in the terminal instead.

## The Loop

1. **Apply the `artifact-design` skill** before writing the page — it carries the design process and taste guidance that make a mockup look intentional rather than templated.

2. **Write a self-contained HTML file** to your session scratchpad directory with a semantic name (`layout.html`, `nav-structure.html`, `dashboard-mockup.html`):
   - Write the page content directly — no `<!DOCTYPE>`, `<html>`, `<head>`, or `<body>` tags; the Artifact wrapper adds those.
   - Set a concise, stable `<title>` — it names the artifact in the gallery and tab.
   - Strict CSP: inline all CSS and JS, embed images as `data:` URIs. No external hosts, fonts, CDNs, or fetch/XHR.
   - Responsive: relative units, `max-width:100%` on images, wide content scrolls inside its own container.
   - Use the Write tool, never `cat`/heredoc (dumps noise into the terminal).

3. **Publish with the `Artifact` tool** — pass the file path and a `favicon` (one or two emoji; keep it stable across redeploys of the same mockup). It returns a `claude.ai` URL.

4. **End your turn**: link the URL, give a one-line summary of what's on screen ("Three homepage layouts — single-column, sidebar, and split"), and ask the user to look and reply in the terminal.

5. **Iterate on the same URL**: to revise, edit the *same file* and call `Artifact` with the *same file path* — it redeploys to the same URL. A new file path mints a new URL, so only do that for a genuinely separate mockup.

6. Move to the next question only once the current visual is validated.

## Design Tips

- **2-4 options max** per page. More than that and the comparison blurs.
- **Scale fidelity to the question** — wireframes for layout decisions, polish only for polish questions.
- **Explain the question on the page** — "Which layout feels more professional?", not just "Pick one."
- **Use real content when it matters** — for a data table, show plausible rows; placeholder lorem obscures real density and overflow issues.
- **Keep mockups simple** — structure and hierarchy over pixel perfection, unless the question is about polish.

## Persistence

The published URL persists on `claude.ai` under your org (retention is admin-configurable) and is listed in your gallery - it's the durable "approved design" reference a reviewer can open after the session ends. The local HTML in the scratchpad is just the editable source for iterating; it lives outside the project and needs no gitignore entry.
