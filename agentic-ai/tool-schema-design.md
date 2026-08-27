# Designing Tool Schemas for Reliable Calling

A tool schema is the only interface between an agent's reasoning and the outside world — every action in the [agent loop](agent-loop.md) happens through one. It's easy to treat a schema as an implementation detail (whatever shape the underlying function already has) instead of as what it actually is: documentation read by a model under uncertainty, at the exact moment it's deciding what to do next. A compiler rejects an ambiguous API with an error. A model facing an ambiguous tool schema doesn't error — it picks the most plausible interpretation and proceeds, confidently, on a guess.

## The description is a routing decision, not documentation

A tool's description is evaluated at the moment the model is choosing *whether to call this tool at all*, and *which one* if several look similar. Write it to answer that question specifically — "call this when X" — not to explain what happens internally once it's called. Two tools with descriptions like "manages user data" and "handles user records" are functionally indistinguishable to whatever is deciding between them, even if their implementations are completely different. Naming what each one is *for*, and ideally what it's explicitly *not* for, does more work than a more thorough account of what it does.

## Narrow the parameter space wherever the domain is closed

A single freeform string parameter accepts anything, which means it constrains nothing — the model has to infer the expected shape from the description alone, and small deviations (a different date format, a synonym for an enum value) become runtime failures instead of schema-time impossibilities. Where the valid values are known ahead of time, an enum eliminates a whole category of malformed calls before they happen. Where a parameter is optional, marking it optional deliberately — not just omitting it from `required` by default — tells the model when it's safe to leave something out versus when a call without it will fail.

Strict schemas (rejecting unknown properties rather than silently ignoring them) turn a typo'd parameter name into an immediate, catchable error instead of a call that silently does the wrong thing because the intended parameter was never actually set.

## One tool, one job

A tool with a `mode` parameter that switches its behavior between several unrelated operations is really several tools wearing one name. The model has to hold the mapping from mode to behavior in mind on top of everything else, and a wrong mode selection produces a call that's schema-valid but semantically wrong — which is harder to catch than a call that fails outright. Splitting it into separate tools with separate, specific descriptions moves that selection problem to where it's actually easiest to solve: the routing decision the model is already making before every call.

This isn't a rule against a tool having several related parameters that shape *how* it does one job — it's specifically about parameters that change *what job it's doing*.

## Say what a tool is not for, too

When two tools are adjacent but distinct — a narrow lookup versus a broad search, a fast/cheap path versus a slow/thorough one — the description that prevents misuse is often the negative case, not the positive one: "use this for an exact known identifier; for anything requiring a search across multiple candidates, use the other tool instead." Without that explicit boundary, the two tools' positive descriptions can both sound applicable to the same request, and the model breaks the tie arbitrarily.

## Errors are part of the schema's job, not an afterthought

A failed call needs to tell the next planning step *why* it failed and what a valid call would look like — "invalid input" gives the model nothing to correct; "expected an ISO-8601 date, got `next tuesday`" gives it something to fix immediately. An opaque error code forces either a blind retry with the same arguments (this is the "thrashing" failure mode from the [agent loop](agent-loop.md)) or a separate lookup just to interpret the failure. The error message *is* the interface at the moment something goes wrong — it deserves the same care as the parameter descriptions that prevent the failure in the first place.

## Defer the schema, not just the call, when the tool surface is large

Loading every available tool's full schema into context before the loop even starts has a fixed token cost that scales with how many tools exist, independent of how many will actually be used in a given session. A pattern that avoids this: list tools by name and a one-line description up front, and fetch a tool's complete schema only once something in the task indicates it's actually needed. This keeps the fixed cost proportional to the tool surface a *task* touches rather than the tool surface that *exists* — the same context-growth tradeoff described in the agent loop entry, applied to the tool definitions themselves rather than their outputs.

## Treat a shipped schema like a public API

Once a tool's parameter shape is in use — by a skill, a saved prompt, a habit the model has learned during a long session — changing it breaks every existing caller that assumed the old shape, silently, until something fails downstream. Adding an optional parameter is safe. Renaming a parameter, changing what a value means, or narrowing what used to be accepted is not something to do casually once the tool is in active use.

## Anti-patterns

- **The kitchen-sink tool** — twenty optional parameters covering every feature a resource supports, because it was easier to expose the whole underlying API than to design a task-shaped interface. Every added parameter is one more thing the model has to correctly ignore on every call that doesn't need it.
- **Near-duplicate tools** — two tools whose descriptions overlap enough that the choice between them is effectively arbitrary. If a human reading only the two descriptions can't confidently say which to use, the model can't either.
- **Silent coercion** — accepting a string where a number was intended, or quietly clamping an out-of-range value instead of rejecting it. This produces inconsistent behavior that's much harder to debug than a hard failure at the boundary.
- **No feedback loop** — never looking at which calls actually fail or get misrouted in practice. Schema design is not a one-time exercise; the calls a model actually makes are the ground truth for whether a description was clear.

## Related

- [agent-loop.md](agent-loop.md) — the loop that tool calls are embedded in, including the context-growth and thrashing failure modes this entry expands on.
- [tools-and-skills/claude-code-skills-and-subagents.md](../tools-and-skills/claude-code-skills-and-subagents.md) — a skill's description follows the same "routing signal, not documentation" principle as a tool's.
