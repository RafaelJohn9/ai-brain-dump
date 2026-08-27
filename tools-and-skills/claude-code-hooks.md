# Claude Code Hooks

A hook is a shell command the harness runs automatically in response to a specific event — before or after a tool call, at session start or stop, and similar points in the session lifecycle. Hooks are configured by the user or the project, not chosen by the agent, and that's the entire point: a hook runs every time its trigger fires, regardless of what the agent is currently focused on, what it's already decided, or whether it remembered an instruction from earlier in a long session.

## The problem repo-level instructions can't fully solve

[repo-level-agent-instructions.md](repo-level-agent-instructions.md) covers `CLAUDE.md`/`AGENTS.md`: durable, repo-specific context an agent reads at the start of a session. That's *guidance* — the agent is expected to follow it, and generally will, but "expected to follow" is a probabilistic guarantee, not a mechanical one. An instruction sitting in a file competes for attention with everything else in a long session; under enough context pressure, or in an edge case the instruction's author didn't anticipate, it can get missed.

A hook doesn't compete for attention — it's not something the agent has to remember to apply, because it isn't the agent applying it at all. It's the harness itself intercepting an event and running a command, deterministically, whether or not the agent's current reasoning would have arrived at the same conclusion on its own.

## What hooks are actually used for

- **Blocking a specific dangerous action before it happens** — a hook that inspects a command about to run and refuses it (a destructive `rm -rf`, a force-push to a protected branch) removes the need to trust that the agent's in-the-moment judgment catches every case, every time.
- **Mechanically enforcing style or process** — running a formatter or linter automatically after a file edit means code style is enforced as a fact about the repository, not as a request the agent has to remember to honor on every single change.
- **Logging and audit trail** — recording what actions were taken, for review later, independent of whatever summary the agent itself might give of its own actions.
- **Organizational policy** — a rule like "no direct commits to the default branch" enforced at the mechanical level doesn't depend on every agent session, or every person's session, independently knowing and choosing to respect that rule.

The common thread: a hook is the right tool specifically when a rule needs to hold *unconditionally*, not just *usually*.

## The trust model

Hook output is treated as coming from the user, not as an arbitrary tool result to weigh against other considerations. This matters in practice: when a hook blocks an action and explains why, that explanation should actually change what happens next — the agent should adjust its approach, the same way it would if a person had just said "don't do that, here's why" — rather than being logged and set aside as one input among several.

That trust model puts real weight on a hook's failure message. A hook that blocks silently, or with an opaque error, forces exactly the kind of blind-retry-or-dead-end situation described in [agentic-ai/tool-schema-design.md](../agentic-ai/tool-schema-design.md): a failure with no information about why or what to do differently doesn't correct behavior, it just stops it. A hook that explains *why* it blocked something gives the next step somewhere to go.

## Hooks as an external termination signal

[agentic-ai/agent-loop.md](../agentic-ai/agent-loop.md) names three ways a loop can know to stop: the model deciding, a turn budget, or an external signal — and calls the external signal the most reliable of the three, because it doesn't depend on the model correctly self-assessing. A hook that blocks a specific action is exactly this kind of external signal, scoped to one decision point rather than the whole loop: it doesn't ask the model to remember not to do something, it makes the something not happen.

## Pitfalls

- **Firing too broadly.** A hook that blocks or intercepts things that were actually fine trains whoever's working with it to route around the hook rather than respect it — the same failure mode as an over-eager validation layer described in [coding-patterns/error-handling-boundaries.md](../coding-patterns/error-handling-boundaries.md), applied to a process control instead of a code path. A hook with a high false-positive rate is worse than no hook, because it teaches people to stop trusting its blocks even when a block is actually correct.
- **Failing without a clear message.** Covered above, but worth restating as a pitfall on its own: a hook is only as useful as its failure is legible.
- **Duplicating what the instructions file already says.** A rule stated in prose in `CLAUDE.md` *and* separately enforced by a hook is two sources of truth for one rule, which can drift when one gets updated and the other doesn't. The rule of thumb: enforce a rule with a hook, or state it as guidance — not both redundantly for the same rule, unless the prose version is carrying information the hook's block message genuinely can't (the *why* behind the rule, which matters for judgment calls the hook itself doesn't have to make).

## Related

- [repo-level-agent-instructions.md](repo-level-agent-instructions.md) — the guidance-level counterpart: durable context the agent is expected to apply, as opposed to a rule mechanically enforced regardless of whether it's applied.
- [agentic-ai/agent-loop.md](../agentic-ai/agent-loop.md) — external signals as the most reliable class of loop-termination condition; a hook is that principle applied to a single decision point.
- [agentic-ai/tool-schema-design.md](../agentic-ai/tool-schema-design.md) — the same "an error should say what went wrong and what to do differently" argument, here applied to a hook's block message instead of a tool's error output.
