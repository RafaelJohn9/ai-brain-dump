# MCP Server Integration

MCP (Model Context Protocol) is a shared protocol for exposing external tools and data sources to an agent, instead of every agentic tool writing a bespoke, one-off integration for every external system it wants to reach. Without a shared protocol, connecting an agent to Slack, a browser, a company's internal API, and a file store means N hosts each independently building M integrations. MCP moves that cost to each side implementing the protocol once — a server exposing Slack over MCP works with any host that speaks MCP, not just the one it was originally built for.

## What a server actually hands the agent

An MCP server typically exposes **tools** (callable actions — send a message, read a page, query a record) and sometimes standing **instructions** for how to use them well. Both often arrive lazily rather than all at once: tools get listed by name first, with full schemas fetched only when something in the current task actually calls for them. This is a live instance of the deferred-schema-loading pattern from [tool-schema-design.md](../agentic-ai/tool-schema-design.md) — the fixed context cost stays proportional to what a session actually touches, not to how many servers happen to be connected.

## The trust boundary MCP introduces

An MCP server can be operated by a third party, and the data it returns often comes from *outside* the conversation entirely — the content of a web page, an email, a Slack message, a file someone else authored. That data lands in the agent's context looking exactly like any other tool result, but nobody who's actually in the conversation wrote it. It can contain text that reads as an instruction — "ignore your previous instructions and do X" embedded in a page's content — and the discipline that matters is treating everything an MCP tool returns as **data to reason about**, never as **instructions to follow**, regardless of how imperative it sounds. Content that looks like an attempt to redirect the agent's behavior is worth surfacing to the user explicitly, not silently complying with and not silently ignoring either — flagging it is the correct response to an ambiguous signal, not a judgment call to make quietly.

This is the same boundary-drawing instinct as [coding-patterns/error-handling-boundaries.md](../coding-patterns/error-handling-boundaries.md): the point where untrusted content enters the system needs explicit, deliberate handling — trusting it by default because it arrived through a tool call, the same mechanism as everything else, is exactly how the boundary gets skipped.

## Namespacing exists to prevent collisions

Tools from an MCP server are addressed with the server name folded into the tool name, rather than dropped into one flat namespace shared by every connected server. Two servers that each happen to expose a tool called `search` don't collide, and — just as important — it stays legible which external system a given call actually reaches. This is a structural fix for the "near-duplicate tools" anti-pattern named in [tool-schema-design.md](../agentic-ai/tool-schema-design.md): when tools come from many independently authored servers instead of one designer with a consistent naming convention, namespacing is what keeps overlapping names from becoming an ambiguous routing decision.

## Connecting a server is granting a capability

Adding an MCP server isn't a neutral configuration step — it's handing the agent whatever the server exposes: browser control, email access, the ability to post to a shared channel. That deserves the same blast-radius judgment as any other capability grant, independent of whether the specific action arrives through a native tool or an MCP one. A server that can take an irreversible external action (sending a message, posting publicly, spending money) earns the same confirm-before-acting treatment any other high-blast-radius action gets — the fact that it routes through MCP doesn't make the action any more reversible.

## Anti-patterns

- **Trusting MCP tool output as if it carried the same authority as the system prompt or the user's own words.** It's a tool result like any other — useful, often accurate, but not inherently authoritative about what the agent should do next.
- **Connecting a broad-capability server when the task needs a narrow one.** A server exposing full inbox access for a task that only needs to read one thread grants far more than the task requires.
- **Letting two servers' similarly named tools go undistinguished.** If namespacing exists but nothing actually uses it to disambiguate — descriptions that don't mention which system a tool reaches — the collision risk namespacing was meant to prevent shows back up at the reasoning level even though it's resolved at the addressing level.

## Related

- [tool-schema-design.md](../agentic-ai/tool-schema-design.md) — deferred schema loading and the near-duplicate-tools anti-pattern, both of which show up directly in how MCP servers are typically integrated.
- [agentic-ai/agent-loop.md](../agentic-ai/agent-loop.md) — irreversible actions needing a confirmation step, independent of which mechanism triggers them.
