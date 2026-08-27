# Roadmap

What this repo is for and what's actually in it right now, vs. what's just scaffolded.

## Goal

A real, usable resource on: building agentic AI systems, UX/UI design, architectural patterns, coding patterns, and the tools/skills that make an AI-assisted workflow actually work. Not a link dump — each entry should teach something a reader couldn't get from a five-minute search.

## Status by section

| Section | Status | Notes |
|---|---|---|
| [prompts/](prompts/) | populated | 8 system prompts, indexed |
| [research/](research/) | populated | LLM landscape snapshot + document-processing deep dive |
| [agentic-ai/](agentic-ai/) | growing | 1 entry: the agent loop |
| [architecture-patterns/](architecture-patterns/) | growing | 1 entry: RAG pipeline architecture |
| [coding-patterns/](coding-patterns/) | growing | 1 entry: writing agent-legible code |
| [ux-ui-design/](ux-ui-design/) | growing | 1 entry: avoiding the AI-generated look |
| [tools-and-skills/](tools-and-skills/) | growing | 1 entry: Claude Code skills vs. subagents |

Every section now has at least one real entry — the priority shifts from "seed every section" to depth and upkeep.

## Near-term priorities

1. Add a second entry to each section rather than starting new sections — depth over breadth still holds.
2. Re-verify `research/llm-model-landscape.md` periodically — it's a dated snapshot (last updated ~September 2025) and will rot.
3. Candidate next entries: tool-schema design (`agentic-ai/`), design tokens/systems (`ux-ui-design/`), error-handling patterns (`coding-patterns/`), service-boundary patterns (`architecture-patterns/`).

## Conventions

- Each top-level section has a `README.md` stating its scope. Keep it current if scope shifts.
- Prefer one focused file per topic over one giant file per section.
- No filler: an entry restating well-known basics without a concrete example, comparison, or opinion isn't worth adding.
