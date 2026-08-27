# Agentic AI

Notes on building agents: things that plan, call tools, hold state across steps, and recover when a step fails — as opposed to a single prompt-in/completion-out call.

**Status: seed.** This section is scaffolded but not yet populated. See the [root ROADMAP](../ROADMAP.md) for what's planned.

## Scope

- **Agent loops** — plan/act/observe cycles, ReAct-style reasoning, when to stop.
- **Tool use** — designing tool schemas an LLM can call reliably, error surfaces, retries.
- **Multi-agent systems** — orchestrator/worker patterns, when multiple agents beat one, handoff protocols.
- **Memory** — what to persist across turns/sessions vs. what belongs in a single context window.
- **Evaluation** — how to tell if an agent is actually working, not just producing plausible-looking output.
- **Failure modes** — infinite loops, tool misuse, context poisoning, and how to guard against them.

## Related

- [prompts/](../prompts/) has ready-to-use system prompts for individual agent roles (research agent, prompt interpreter, etc.) — this section is where the surrounding *system design* around those prompts belongs.
