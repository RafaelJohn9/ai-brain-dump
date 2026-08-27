# Agentic AI

Notes on building agents: things that plan, call tools, hold state across steps, and recover when a step fails — as opposed to a single prompt-in/completion-out call.

**Status: growing.** See the [root ROADMAP](../ROADMAP.md) for what's planned.

## Contents

- [agent-loop.md](agent-loop.md) — the plan/act/observe loop, termination conditions, context growth as the loop's real resource constraint, tool-call design, and common failure modes (thrashing, premature stopping, context poisoning, runaway autonomy).
- [tool-schema-design.md](tool-schema-design.md) — writing tool schemas the model can call reliably: description as a routing signal, narrowing parameters, one tool/one job, actionable errors, deferred schema loading.
- [agent-memory.md](agent-memory.md) — what's worth persisting across sessions vs. re-deriving from current state, why the reasoning behind a fact matters more than the fact itself, treating a memory as a checkable claim rather than a guarantee, and an index layer for stores that grow over time.

## Scope

- **Agent loops** — plan/act/observe cycles, ReAct-style reasoning, when to stop.
- **Tool use** — designing tool schemas an LLM can call reliably, error surfaces, retries.
- **Multi-agent systems** — orchestrator/worker patterns, when multiple agents beat one, handoff protocols.
- **Memory** — what to persist across turns/sessions vs. what belongs in a single context window.
- **Evaluation** — how to tell if an agent is actually working, not just producing plausible-looking output.
- **Failure modes** — infinite loops, tool misuse, context poisoning, and how to guard against them.

## Related

- [prompts/](../prompts/) has ready-to-use system prompts for individual agent roles (research agent, prompt interpreter, etc.) — this section is where the surrounding *system design* around those prompts belongs.
