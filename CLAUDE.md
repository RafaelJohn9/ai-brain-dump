# Repo Instructions

A personal, in-the-open knowledge base on agentic AI, UX/UI design, architecture patterns, coding patterns, and the tools/skills around an AI-assisted workflow. Read [ROADMAP.md](ROADMAP.md) first for current status; each top-level directory has its own `README.md` stating scope.

## Adding an entry

1. One focused topic per file, kebab-case filename, in the relevant section directory (`agentic-ai/`, `architecture-patterns/`, `coding-patterns/`, `ux-ui-design/`, `tools-and-skills/`).
2. Add a row to that section's `README.md` "Contents" list, in the same one-line-description format as the existing rows.
3. Update `ROADMAP.md`'s status table (entry count and names) and the "Near-term priorities" section.
4. Look for genuine cross-links to related existing entries and add a "Related" section at the bottom — a link that doesn't reflect a real conceptual connection is padding, skip it.
5. Verify every relative markdown link in the file you touched actually resolves before considering it done. A wrong `../` prefix is the most common mistake here and won't surface until someone clicks it.

## Writing bar

No filler: an entry needs a concrete example, comparison, or opinion backed by a reason — not a restatement of well-known basics. Prefer several focused files over one file covering multiple topics.

## Visual artifacts (mainly ux-ui-design/)

Default is markdown-only. Pair an entry with a published visual Artifact only when the topic is genuinely visual (a color palette, a layout) and reading hex codes or prose wouldn't convey it as well as seeing it rendered. If you publish one:

- Commit the `.html` source to the repo alongside the markdown entry — not just the hosted link.
- Never link between two published artifacts with a relative path (`other-page.html`). Each artifact is served from its own isolated origin, so that link is dead on the live page. Link the full `https://claude.ai/code/artifact/...` URL instead.

## Git

One commit per entry, not batched, with a message describing what the entry actually covers. Do not add an AI co-author line to commits in this repo.
