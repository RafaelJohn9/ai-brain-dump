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
| [tools-and-skills/](tools-and-skills/) | growing | 1 entry: Claude Code skills vs. subagents |

Every section has at least one real entry; four of five now have two. The cross-links between entries (e.g. tool-schema-design ↔ agent-loop, error-handling-boundaries ↔ agent-legible-code) are starting to do real work — keep adding them as new entries land.

## Near-term priorities

1. `tools-and-skills/` is the one section still at a single entry — candidate next: a real MCP server integration write-up, or a second Claude Code pattern (hooks, or the CLAUDE.md/AGENTS.md convention referenced from `coding-patterns/agent-legible-code.md`).
2. Re-verify `research/llm-model-landscape.md` periodically — it's a dated snapshot (last updated ~September 2025) and will rot.
3. Keep depth over breadth: a third entry per section beats a sixth section.

## Conventions

- Each top-level section has a `README.md` stating its scope. Keep it current if scope shifts.
- Prefer one focused file per topic over one giant file per section.
- No filler: an entry restating well-known basics without a concrete example, comparison, or opinion isn't worth adding.
