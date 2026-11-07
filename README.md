# Craft

Skills for developing features continuously across sessions, instead of treating each task as a one-off build.

- **`craft:brainstorm`** is a design companion. It briefs you on how the area works today, what the request changes, and which decisions and gaps are open. Then it helps you decide through discussion, audits, comparisons, and mockups, and writes a dated spec once the decisions settle.
- **`craft:worktree-cleanup`** removes every local worktree and branch whose work already exists on the remote, in one pass. It checks merged branches, remote branches, pull request heads, and squashed or rebased copies, and keeps uncommitted, ignored, and live work.

## Feature memory

Each feature keeps one small `overview.md` of decisions that outlive the code, plus the specs that produced them. A new session starts from that file instead of re-deriving what was already decided.

```
~/.claude/projects/<project-slug>/craft/memory/
├── specs/YYYY-MM-DD-<topic>-design.md
└── features/<feature>/overview.md            # a hub holds <sub-feature>/overview.md
```

`<project-slug>` is the project's absolute path with `/` replaced by `-`, the same encoding Claude Code uses. Memory lives outside the working tree, so every worktree shares it and it never shows up in `git status`. Full rules: [`references/memory.md`](references/memory.md).

## Installation

Install from this repo via the Claude Code plugin marketplace. In a Claude Code session:

```
/plugin marketplace add rail-chamidullin-myriad/craft
/plugin install craft@craft
```

The marketplace is named `craft` and publishes one plugin also named `craft` - hence `craft@craft`.

Manage:

```
/plugin list
/plugin marketplace update craft     # pull newer versions
/plugin uninstall craft@craft        # remove
```

### Local development install

If you're hacking on craft itself, point the marketplace at your local checkout:

```
/plugin marketplace add ./
/plugin install craft@craft
```

Re-run `/plugin marketplace update craft` after edits to pick up changes.

## Origin

Craft started as a fork of [Superpowers](https://github.com/obra/superpowers) and went its own direction once feature memory became the central concept. The spec template and the design-review habits come from Superpowers (see `CREDITS.md`).

## License

Personal workflow skills. Use, adapt, ignore - whatever fits your flow.
