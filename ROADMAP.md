# Roadmap

What this repo is for and what's actually in it right now, vs. what's just scaffolded.

## Goal

A real, usable resource on: building agentic AI systems, UX/UI design, architectural patterns, coding patterns, and the tools/skills that make an AI-assisted workflow actually work. Not a link dump — each entry should teach something a reader couldn't get from a five-minute search.

## Status by section

| Section | Status | Notes |
|---|---|---|
| [prompts/](prompts/) | populated | 8 system prompts, indexed |
| [research/](research/) | populated | LLM landscape snapshot + document-processing deep dive |
| [agentic-ai/](agentic-ai/) | growing | 2 entries: the agent loop, tool-schema design |
| [architecture-patterns/](architecture-patterns/) | growing | 2 entries: RAG pipeline architecture, service boundaries |
| [coding-patterns/](coding-patterns/) | growing | 2 entries: agent-legible code, error-handling boundaries |
| [ux-ui-design/](ux-ui-design/) | growing | 2 entries: avoiding the AI-generated look, design tokens & theming |
| [tools-and-skills/](tools-and-skills/) | growing | 2 entries: Claude Code skills vs. subagents, repo-level agent instructions |

Every section now has at least two real entries. The cross-links between entries (tool-schema-design ↔ agent-loop, error-handling-boundaries ↔ agent-legible-code, repo-level-agent-instructions ↔ agent-legible-code) are starting to do real work — keep adding them as new entries land.

## Near-term priorities

1. Depth over breadth continues: a third entry per section beats a sixth section. No section is urgently behind the others anymore, so the next entry can follow whatever's actually useful rather than filling a gap.
2. Candidate topics not yet covered: memory/evaluation (`agentic-ai/`), hooks or MCP integration (`tools-and-skills/`), a layering/hexagonal-architecture write-up (`architecture-patterns/`), testing patterns (`coding-patterns/`), interaction/empty-state patterns (`ux-ui-design/`).
3. Re-verify `research/llm-model-landscape.md` periodically — it's a dated snapshot (last updated ~September 2025) and will rot.

## Conventions

- Each top-level section has a `README.md` stating its scope. Keep it current if scope shifts.
- Prefer one focused file per topic over one giant file per section.
- No filler: an entry restating well-known basics without a concrete example, comparison, or opinion isn't worth adding.
