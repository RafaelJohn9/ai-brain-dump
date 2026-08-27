# The Agent Loop

The core control-flow pattern behind every coding agent, research agent, or "does things on my behalf" system: a repeating plan → act → observe cycle, instead of one prompt producing one answer. Everything else in this section — multi-agent orchestration, memory, evaluation — is built on top of this loop; it's worth understanding on its own before the more complex patterns.

## The shape

```
   ┌─────────────────────────────────────┐
   │                                       │
   ▼                                       │
Plan (decide next step) ──► Act (call a tool) ──► Observe (read the result)
   │                                                       │
   └──────────────────── loop until done ◄─────────────────┘
```

Concretely, on each turn: the model reads the conversation and tool results so far, decides whether it has enough to answer or needs to take an action, and if it needs an action, emits a structured tool call instead of prose. The caller executes that tool call outside the model, feeds the result back in, and the loop repeats. This is the "ReAct" (reason + act) pattern — the reasoning and the acting are interleaved turn by turn, not planned once upfront and executed blind.

The single-completion case (ask a question, get an answer, done) is the degenerate form of this loop where it runs zero times. What makes something "agentic" isn't the model — it's that the loop can run more than once, informed by what actually happened last time, not just what was planned.

## Termination — the part that's easy to get wrong

A loop needs a clear stopping condition, and there are three common ones, each with a different failure mode:

- **The model decides it's done** (no more tool calls, or an explicit "finish" signal) — the normal case. Fails when the model is either overconfident (stops before verifying) or underconfident (keeps taking unnecessary actions to double-check something already established).
- **A turn/step budget** — a hard ceiling as a backstop against runaway loops. Necessary as a safety net, but a loop that regularly hits its budget is a sign the task decomposition or tool design is wrong, not that the budget should just be raised.
- **An external signal** — a test suite passing, a human approving, a condition in the environment becoming true. The most reliable stopping condition when it's available, because it doesn't depend on the model correctly self-assessing.

Production agent loops usually combine at least two of these: model self-assessment as the primary signal, a budget as the backstop.

## Context growth is the loop's real resource constraint

Every act/observe cycle adds the tool call and its result to the context the model reads next turn. A loop that runs for a while accumulates raw tool output — file contents, search results, command output — most of which is only useful for the one decision it informed, not for every decision after. Left unmanaged, this either blows the context budget or (more insidiously) buries the actually-relevant information under noise the model has to re-read every turn.

The practical mitigations, roughly in order of how often they apply:

- **Discard eagerly.** Once a tool result has answered the question it was called for, nothing requires keeping the raw output around — only the conclusion drawn from it needs to persist forward.
- **Isolate exploratory work.** Delegate open-ended search/exploration to a subagent that keeps its own raw output out of the parent's context and returns only a synthesized result. See [tools-and-skills/claude-code-skills-and-subagents.md](../tools-and-skills/claude-code-skills-and-subagents.md) for this pattern in more detail — it's the same context-growth problem, from the subagent side.
- **Summarize/compact.** When a session runs long regardless, compress older turns into a summary and keep only recent turns verbatim. This trades some fidelity for headroom, and works best when the compression step preserves *decisions made* over *process followed*.
- **Defer tool definitions, not just tool output.** In a system with a large tool surface, loading every tool's full schema into context up front costs tokens before the loop even starts. Listing tools by name only, and fetching a schema on demand right before it's needed, keeps that fixed cost from scaling with the size of the tool library.

## Tool calls are the loop's only interface to the world

Everything the loop can do is bounded by what tools it has and how well those tools are specified. A few things matter more here than they first appear to:

- **A tool's description is a routing signal, not documentation for a human.** The model decides whether and how to call a tool based on that description alone at the moment of the decision — vague or overlapping descriptions between two similar tools produce inconsistent choices between them.
- **Tool failure needs to be legible, not just returned.** An error message that tells the model *why* a call failed (bad argument, permission denied, resource not found) lets the next planning step correct course; a bare failure just invites a retry of the same mistake.
- **Not every action should be autonomous.** Irreversible or high-blast-radius actions (deleting data, sending something externally, spending money) belong behind an explicit confirmation step in the loop, not inside the same unattended act/observe cycle as a read-only lookup. The loop's autonomy and the action's reversibility are two different dials — conflating them is how "the agent did something destructive without asking" incidents happen.

## Failure modes

- **Thrashing** — the loop repeats a similar action expecting a different result, usually because the observation from the failed attempt didn't actually change the model's plan. Often a sign the tool's error output isn't informative enough to correct course.
- **Premature stopping** — the model reports success without having actually verified the outcome (e.g., claiming a fix works without running the test that would confirm it). Mitigated by making verification itself a required tool call in the loop, not an assumption.
- **Context poisoning** — an early wrong assumption or a bad tool result stays in context and keeps influencing every later step, even after later evidence contradicts it. Harder to detect than it sounds, because the model doesn't flag its own stale premises.
- **Runaway autonomy** — the loop keeps taking actions past the point where a human should have been checked in with, because nothing in the loop's design forces a pause. This is a design gap (no confirmation step for the right class of action), not a one-off model mistake.

## Related

- [tools-and-skills/claude-code-skills-and-subagents.md](../tools-and-skills/claude-code-skills-and-subagents.md) — subagents as a way to isolate a chunk of loop work and keep its context cost out of the parent loop.
- [architecture-patterns/rag-pipeline.md](../architecture-patterns/rag-pipeline.md) — retrieval as one tool call among several inside a loop like this one, once it stops being a fixed pipeline stage (see that entry's "Agentic RAG" variant).
