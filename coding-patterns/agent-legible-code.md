# Writing Code an Agent Can Safely Modify

Code that's easy for a human to read isn't automatically code that's easy for an AI coding agent to *change correctly*. The two overlap a lot — clarity helps both — but an agent has different failure modes than a human reader, and a codebase can be optimized for them specifically. This is the coding-pattern flip side of [agentic-ai/agent-loop.md](../agentic-ai/agent-loop.md): that entry covers the loop's own failure modes (thrashing, premature stopping); this one covers how the *code itself* can make those failure modes more or less likely.

## Why this is a distinct concern

A human making a change usually holds a broad, if fuzzy, mental model of the whole system before touching anything. An agent working through a tool-call loop builds a much narrower, more local picture — it reads what it needs for the task in front of it, not the whole call graph. That difference has concrete consequences:

- A change that's obviously safe with full system context can be silently unsafe when made with only local context.
- An agent won't always notice it's missing context the way a human would pause and go look something up — it acts on what it has.
- The cost of a wrong assumption compounds across a loop instead of being caught once, because a wrong early conclusion stays in context and keeps informing later steps (see "context poisoning" in the agent-loop entry).

The patterns below reduce how much any single change depends on context that isn't right there in front of the agent making it.

## Patterns

**Locality of change.** A fix or feature should touch as few files, and as small a diff, as the task actually requires. Small diffs are easier for an agent to reason about correctly in one pass, and easier for whoever reviews the result — human or another agent — to verify actually did what it claims.

**Explicit boundaries and contracts.** A function or module with a clear, narrow interface limits the blast radius of a change to that interface's callers. This matters more for agent-modified code than human-modified code specifically because an agent is less likely to have manually traced every call site before making a change — a clear contract is what lets it *not* have to.

**Self-verification as part of the change, not after it.** A change isn't done when the code compiles or looks right — it's done when something concrete confirms the behavior (a test run, a type check, the feature exercised). Baking a verification step into the definition of "finished" directly targets the premature-stopping failure mode: an agent that's required to run the check before claiming success can't paper over "I believe this works" with "I verified this works."

**Comments carry the *why*, not the *what*.** Both a human and an agent can already read what well-named code does — a comment restating that adds nothing and goes stale the moment the code changes without the comment being updated. What's worth writing down is the part that *isn't* in the code: a non-obvious constraint, a workaround for a specific bug, a reason a seemingly-simpler approach doesn't work. That's information a future reader (human or agent) can't recover by reading the code harder.

**No speculative abstraction.** Three similar call sites are better left as three similar call sites than collapsed into one shared abstraction "in case" a fourth shows up. An abstraction bundles unrelated callers together — changing it to serve one caller's new need risks silently breaking another. That risk is worse for agent-driven changes than deliberate human refactors, because an agent modifying the abstraction for a local task is exactly the situation where the other callers are least likely to be in view.

**Validate at boundaries, trust internally.** Input validation and error handling belong where untrusted data enters the system (user input, an external API response) — not sprinkled defensively through internal code paths that can't actually receive bad data. Without this discipline stated explicitly, an agent asked to "make this more robust" will often add handling for states that can't occur, which adds surface area without adding safety.

**Repo-level instructions as the source of team convention.** A file the agent reads at the start of a session (the `CLAUDE.md` / `AGENTS.md` convention) — naming style, test commands, what's off-limits, architectural decisions already made — means those conventions get applied consistently instead of an agent inferring (and occasionally guessing wrong) a style from whatever files happen to be nearby.

## Anti-patterns

- **Narrating the diff in comments** — `// fixed to handle the null case from issue #123` describes the *change*, not the code; it's already in the commit history and rots the moment the surrounding code moves on.
- **Backwards-compatibility shims for a change that didn't need them** — renaming an unused variable to `_var` instead of deleting it, keeping a re-export "just in case," a `// removed` comment marking something that's gone. These accumulate as noise that outlives the reason they were added, and they're a symptom of an agent being unsure it's allowed to actually remove something.
- **Unrequested scope expansion** — cleaning up unrelated code "while already in the area." Bundled with an unrelated task, it makes the diff harder to verify and attributes an unreviewed change to the same commit as a reviewed one.
- **Fabricated robustness** — try/catch blocks, fallback branches, or default values for situations that genuinely cannot occur given the caller's guarantees. Each one is a claim about a failure mode that isn't real, and it costs a future reader the effort of checking whether it might be.

## Related

- [agentic-ai/agent-loop.md](../agentic-ai/agent-loop.md) — the loop-level counterpart: how premature stopping and context poisoning happen in the agent's process, independent of the code itself.
- [tools-and-skills/claude-code-skills-and-subagents.md](../tools-and-skills/claude-code-skills-and-subagents.md) — packaging repo-specific conventions as a skill instead of relying on an agent to infer them fresh each session.
