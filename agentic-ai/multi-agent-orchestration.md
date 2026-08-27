# Multi-Agent Orchestration

Some tasks are better handled by more than one agent working together than by a single continuous loop — but "more than one agent" is a coordination problem the moment it's more than a novelty, and the coordination itself has real design decisions, independent of the mechanism any particular tool uses to actually spawn a second agent. [tools-and-skills/claude-code-skills-and-subagents.md](../tools-and-skills/claude-code-skills-and-subagents.md) covers the mechanics of one such mechanism (Claude Code's subagents and forks); this entry is about the general pattern those mechanics implement.

## The orchestrator/worker topology

One agent — the orchestrator — holds the overall goal, decomposes it into subtasks, dispatches each to a worker, and integrates what comes back. A worker doesn't inherit the orchestrator's accumulated context by default; it starts from whatever brief it's given. That means the dispatch itself has to be a self-contained briefing — the specific facts, file paths, and definition of "done" a worker needs — not a reference to context the worker was never actually given. A worker handed a vague pointer instead of a real brief doesn't fail loudly; it produces plausible-sounding work built on guessed-at context, which is a harder failure to catch than an outright error.

## When multiple agents genuinely beat one

- **Independent subtasks with no data dependency between them.** If subtask B doesn't need anything subtask A produces, dispatching both in parallel saves real wall-clock time; if B does need A's output, parallelizing them just adds coordination overhead for no benefit.
- **A check that needs independence from the thing being checked.** [evaluation.md](evaluation.md)'s "don't let the agent grade its own work uncritically" is a concrete case for a second agent: a reviewer that never saw the implementer's reasoning can catch a mistake the implementer's own self-assessment inherited and missed.
- **Isolating exploratory or high-volume work from a main thread's context.** The same context-growth argument from [agent-loop.md](agent-loop.md) — a worker that does a lot of open-ended searching and returns only a synthesized result keeps that search's raw output from ever entering the orchestrator's context in the first place.

## When it doesn't

A task with a tight sequential dependency chain, where each step genuinely needs the full accumulated context of everything before it, doesn't parallelize — splitting it into separate agent dispatches just adds the overhead of re-establishing context at each handoff, for zero concurrency benefit, since the steps couldn't run concurrently anyway. A single continuous loop is simpler and no slower for that shape of task. Multi-agent orchestration is a tool for a specific problem shape (parallelizable, or benefiting from independence), not a default upgrade over a single loop.

## The handoff is what actually needs care

What crosses from one agent to another shouldn't be the full raw transcript of how the sending agent got there — it should be the specific facts and decisions the receiving agent needs to act correctly. A raw transcript dump is easier to produce but forces the receiving agent to spend its own context re-deriving what mattered from what didn't, which is exactly the cost a synthesized handoff (a short brief, a decision, a file path, a specific finding) is worth the extra effort to construct instead.

## Aggregation is real work

Combining several workers' results isn't a formality — it's where contradictions between two workers' findings, overlapping or duplicated effort, and one worker's output quietly invalidating an assumption another worker's output depended on all actually surface. Treating aggregation as "concatenate the results" produces a combined answer that's only as trustworthy as its weakest, least-scrutinized component — the orchestrator has to actually reconcile what came back, not just stitch it together.

## Choreography vs. orchestration, the same tradeoff seen elsewhere

[architecture-patterns/service-boundaries.md](../architecture-patterns/service-boundaries.md) and [architecture-patterns/rag-pipeline.md](../architecture-patterns/rag-pipeline.md)'s "agentic RAG" variant both gesture at the same underlying choice this pattern makes explicit: a small number of coordinating parties can either have one of them explicitly directing the others (orchestration — visible flow, one place to look, a new component that has to be built and kept correct) or each independently reacting to signals with no central coordinator (choreography — simpler for very few parties, harder to reason about as the count grows). Multi-agent work defaults toward orchestration for exactly the reason most distributed coordination problems do: an explicit sequence that's visible in one place is easier to debug than a shared understanding no single component actually holds.

## Anti-patterns

- **Spawning multiple agents for a task simple enough to do directly.** The generalized version of "spawning a subagent per trivial lookup" from [claude-code-skills-and-subagents.md](../tools-and-skills/claude-code-skills-and-subagents.md) — orchestration overhead paid for a task that never needed splitting.
- **Dispatching without a self-contained brief.** Forces a worker to guess at context it was never actually given, producing plausible work built on an assumption nobody checked.
- **Treating aggregation as concatenation.** Skips the actual work of reconciling contradictions and dependencies between workers' results.

## Related

- [tools-and-skills/claude-code-skills-and-subagents.md](../tools-and-skills/claude-code-skills-and-subagents.md) — the concrete mechanism (forks vs. fresh agents) this pattern is built on in Claude Code specifically.
- [agent-loop.md](agent-loop.md) — the context-growth argument that motivates isolating a worker's exploratory work from the orchestrator's own context.
- [evaluation.md](evaluation.md) — independent grading as one of the strongest motivating cases for using more than one agent.
