# Roadmap

What this repo is for, and what's actually in it right now.

## Goal

A real, usable resource on: building agentic AI systems, UX/UI design, architectural patterns, coding patterns, and the tools/skills that make an AI-assisted workflow actually work. Not a link dump — each entry should teach something a reader couldn't get from a five-minute search.

## Status by section

| Section | Status | Notes |
|---|---|---|
| [prompts/](prompts/) | populated | 8 system prompts, indexed |
| [research/](research/) | populated | LLM landscape snapshot + document-processing deep dive |
| [agentic-ai/](agentic-ai/) | growing | 5 entries: the agent loop, tool-schema design, agent memory, evaluation, multi-agent orchestration |
| [architecture-patterns/](architecture-patterns/) | growing | 5 entries: RAG pipeline architecture, service boundaries, layering & hexagonal architecture, CQRS & event sourcing, sagas |
| [coding-patterns/](coding-patterns/) | growing | 5 entries: agent-legible code, error-handling boundaries, testing patterns, functional patterns, classic patterns in modern languages |
| [ux-ui-design/](ux-ui-design/) | growing | 5 entries: avoiding the AI-generated look, design tokens & theming, color scheme patterns, subject-driven identity (2 visual specimen artifacts), interaction states |
| [tools-and-skills/](tools-and-skills/) | growing | 5 entries: Claude Code skills vs. subagents, repo-level agent instructions, Claude Code hooks, MCP server integration, evaluating prompts and skills |

Every section now sits at exactly 5 entries — the first time the spread has been perfectly even. The cross-link web keeps compounding (agent-memory ↔ agent-loop ↔ tool-schema-design ↔ repo-level-agent-instructions ↔ claude-code-hooks ↔ mcp-server-integration; evaluation ↔ evaluating-prompts-and-skills ↔ agent-loop ↔ testing-patterns ↔ agent-memory; multi-agent-orchestration ↔ claude-code-skills-and-subagents ↔ agent-loop ↔ evaluation; testing-patterns ↔ error-handling-boundaries ↔ agent-legible-code ↔ functional-patterns ↔ classic-patterns-in-modern-languages ↔ layering-and-hexagonal; color-scheme-patterns ↔ design-tokens-and-theming ↔ avoiding-the-ai-generated-look ↔ subject-driven-identity; interaction-states ↔ error-handling-boundaries ↔ avoiding-the-ai-generated-look; layering-and-hexagonal ↔ service-boundaries ↔ cqrs-and-event-sourcing ↔ sagas ↔ multi-agent-orchestration ↔ rag-pipeline) — new entries should keep looking for a genuine link rather than restating a neighboring entry's point.

`evaluating-prompts-and-skills.md` names a real, current gap: none of the 8 templates in `prompts/` ship with example inputs or a golden set. Worth fixing directly rather than just noting — see near-term priorities.

`ux-ui-design/` has two entries paired with a published visual Artifact rather than markdown alone (color-scheme-patterns, subject-driven-identity) — it's clearly the section where "look at it" beats "read about it" most often. The default for new entries elsewhere stays markdown-only unless the topic is inherently visual.

## Near-term priorities

1. Every section is now even at 5 — no section is "behind." Next entries can follow whatever's most useful rather than filling a gap.
2. Concrete, actionable gap from `evaluating-prompts-and-skills.md`: add a small set of example inputs/expected behavior to each of the 8 templates in `prompts/` — currently none of them have one.
3. Candidate topics not yet covered: design-system component libraries — build vs. adopt in practice (`ux-ui-design/`), frontend/component architecture (`architecture-patterns/`), memory/state patterns for stateful UI (`coding-patterns/` or `ux-ui-design/`).
4. Re-verify `research/llm-model-landscape.md` periodically — it's a dated snapshot (last updated ~September 2025) and will rot.
5. If `color-scheme-specimens.html` or `theme-specimens.html` are ever republished with changes, redeploy via the same file path/URL so the links in their companion markdown entries keep working.

## Conventions

- Each top-level section has a `README.md` stating its scope. Keep it current if scope shifts.
- Prefer one focused file per topic over one giant file per section.
- No filler: an entry restating well-known basics without a concrete example, comparison, or opinion isn't worth adding.
