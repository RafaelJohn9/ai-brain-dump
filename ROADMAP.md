# Roadmap

What this repo is for and what's actually in it right now, vs. what's just scaffolded.

## Goal

A real, usable resource on: building agentic AI systems, UX/UI design, architectural patterns, coding patterns, and the tools/skills that make an AI-assisted workflow actually work. Not a link dump — each entry should teach something a reader couldn't get from a five-minute search.

## Status by section

| Section | Status | Notes |
|---|---|---|
| [prompts/](prompts/) | populated | 8 system prompts, indexed |
| [research/](research/) | populated | LLM landscape snapshot + document-processing deep dive |
| [agentic-ai/](agentic-ai/) | growing | 3 entries: the agent loop, tool-schema design, agent memory |
| [architecture-patterns/](architecture-patterns/) | growing | 2 entries: RAG pipeline architecture, service boundaries |
| [coding-patterns/](coding-patterns/) | growing | 3 entries: agent-legible code, error-handling boundaries, testing patterns |
| [ux-ui-design/](ux-ui-design/) | growing | 4 entries: avoiding the AI-generated look, design tokens & theming, color scheme patterns, subject-driven identity (2 visual specimen artifacts) |
| [tools-and-skills/](tools-and-skills/) | growing | 2 entries: Claude Code skills vs. subagents, repo-level agent instructions |

The cross-link web between entries keeps compounding (agent-memory ↔ agent-loop ↔ tool-schema-design ↔ repo-level-agent-instructions; testing-patterns ↔ error-handling-boundaries ↔ agent-legible-code; color-scheme-patterns ↔ design-tokens-and-theming ↔ avoiding-the-ai-generated-look ↔ subject-driven-identity) — new entries should keep looking for a genuine link rather than restating a neighboring entry's point.

`ux-ui-design/` now has two entries paired with a published visual Artifact rather than markdown alone (color-scheme-patterns, subject-driven-identity) — it's clearly the section where "look at it" beats "read about it" most often. The default for new entries elsewhere stays markdown-only unless the topic is inherently visual.

## Near-term priorities

1. `architecture-patterns/` and `tools-and-skills/` are the sections at two entries and now visibly behind `ux-ui-design/`'s four — next passes should even these up.
2. Candidate topics not yet covered: evaluation (`agentic-ai/` — how to tell an agent is actually working, not just producing plausible output), hooks or MCP integration (`tools-and-skills/`), a layering/hexagonal-architecture write-up (`architecture-patterns/`), interaction/empty-state patterns (`ux-ui-design/`).
3. Re-verify `research/llm-model-landscape.md` periodically — it's a dated snapshot (last updated ~September 2025) and will rot.
4. If `color-scheme-specimens.html` or `theme-specimens.html` are ever republished with changes, redeploy via the same file path/URL so the links in their companion markdown entries keep working.

## Conventions

- Each top-level section has a `README.md` stating its scope. Keep it current if scope shifts.
- Prefer one focused file per topic over one giant file per section.
- No filler: an entry restating well-known basics without a concrete example, comparison, or opinion isn't worth adding.
