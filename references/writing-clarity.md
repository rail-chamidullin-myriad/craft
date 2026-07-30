# Writing clarity - reader-facing prose

Applies to anything a human reads: design specs, PR titles and descriptions, commit messages, code-review findings, and text presented in chat.

For internal craft memory (`overview.md`, `changes/*.md`), use the telegraphic [`writing-style.md`](writing-style.md) instead. That style is denser and not meant for review.

Goal: say the thing plainly. The reader skims once and gets it.

## Pattern

`[thing] [action] [reason]. [next step].`

Not: "The reason the component re-renders is that a new object reference is created on each render, so you should probably wrap it."

Yes: "New object ref each render forces a re-render. Wrap it in `useMemo`."

## Rules

- One idea per sentence. Short sentences beat long ones.
- Cut filler and hedging: `just`, `really`, `basically`, `actually`, `simply`, `essentially`, `quite`, `very`.
- Cut pleasantries and preamble: `sure`, `certainly`, `happy to`, `In order to`, `It should be noted that`, `We've been having an issue where`. State the point.
- No noun-stacks or dense engineer-prose. Name the concrete thing.
- Plain word over fancy: `use` not `utilize`, `fix` not `remediate`, `about` not `approximately`, `start` not `initiate`.
- No invented abbreviations (`cfg`, `impl`, `req`, `res`, `fn`). The tokenizer splits them the same as the full word, so you save nothing and the reader still has to decode. The full word is cheaper and clearer.
- Active voice: `This adds X`, not `X is added`.
- Keep technical terms, identifiers, and numbers exact. Plain is not vague. `orphan rows` and `720 MB -> 580 MB` beat `leftover data` and `much smaller`.

## Do not compress these

Terseness that risks a misread is a bug, not a saving. Write these in full, careful sentences:

- Security warnings and their consequences.
- Confirmations before an irreversible action (deletes, overwrites, force-pushes).
- Multi-step sequences where dropped words make the order ambiguous. `migrate table drop column backup first` reads three ways; spell it out.

## Example

Dense:

> Facilitation of the reconciliation process was implemented in order to address the scenario wherein alias-to-index mapping divergence could potentially manifest.

Plain:

> Reconcile the alias to the index at startup so it can't drift.

## Guardrail

Clarity is not compression. Never drop a qualifier that changes meaning. Shorter wins only when it stays exact.
