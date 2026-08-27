# Roadmap

What this repo is for and what's actually in it right now, vs. what's just scaffolded.

## Goal

A real, usable resource on: building agentic AI systems, UX/UI design, architectural patterns, coding patterns, and the tools/skills that make an AI-assisted workflow actually work. Not a link dump — each entry should teach something a reader couldn't get from a five-minute search.

## Status by section

| Section | Status | Notes |
|---|---|---|
| [prompts/](prompts/) | populated | 8 system prompts, indexed |
| [research/](research/) | populated | LLM landscape snapshot + document-processing deep dive |
| [agentic-ai/](agentic-ai/) | seed | scope defined, no entries yet |
| [architecture-patterns/](architecture-patterns/) | seed | scope defined, no entries yet |
| [coding-patterns/](coding-patterns/) | seed | scope defined, no entries yet |
| [ux-ui-design/](ux-ui-design/) | seed | scope defined, no entries yet |
| [tools-and-skills/](tools-and-skills/) | seed | scope defined, no entries yet |

## Near-term priorities

1. Fill in one real entry per seed section before adding more scaffolding — depth over breadth.
2. `tools-and-skills/` — write up the Claude Code skill/subagent pattern this repo already uses as the first entry; it's real, working knowledge, not theory.
3. `architecture-patterns/` — write up RAG pipeline architecture, since `research/document-processing/` already has the raw material to draw from.
4. Re-verify `research/llm-model-landscape.md` periodically — it's a dated snapshot (last updated ~September 2025) and will rot.

## Conventions

- Each top-level section has a `README.md` stating its scope. Keep it current if scope shifts.
- Prefer one focused file per topic over one giant file per section.
- No filler: an entry restating well-known basics without a concrete example, comparison, or opinion isn't worth adding.
