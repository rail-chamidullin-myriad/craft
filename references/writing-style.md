# Writing Style - Telegraphic, Self-Decoding

How every line of craft memory (`overview.md`, `changes/*.md`) is phrased. A memory file is written once and read every future session, so terse phrasing pays on every load.

Applied at two points: the write protocol (`feature-files.md`) phrases each new line this way; the compaction audit (`compaction.md`) rewrites surviving prose into it.

## Transforms

- Fragments over sentences. Drop articles, filler, and the subject when obvious.
- `=` for "is", `->` for "leads to / then", `+` for "and".
- Problem before solution.
- One fact per line.

Examples:

`You should wrap the object in useMemo, since a new reference is created on every render.`
-> `New ref each render. Wrap object in useMemo.`

`Postgres is the source of truth for both search text and embedding; OpenSearch is the derived mirror, re-synced with no re-embed.`
-> `Postgres = SOT for search text + embedding. OpenSearch = derived mirror, re-synced no re-embed.`

## Guardrails

- **Non-lossy.** A constraint keeps any qualifier that changes its meaning. Compress phrasing, never substance.
- **Self-decoding.** Only universally-read connectors (`= -> +`). No invented symbols, no abbreviation glossary. If a reader would need a legend, it is over-compressed.
- **Precision over density for constraints.** An explanation tolerates aggressive trimming; a load-bearing constraint does not.
