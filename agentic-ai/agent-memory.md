# Agent Memory

A single context window ends when the session does. Memory is whatever's designed to survive past that — a fact about a user's preferences that should still apply next week, a correction that shouldn't need to be given twice, a project decision that should keep informing suggestions after the conversation that produced it is gone. It's the cross-session counterpart to the context-growth problem in [agent-loop.md](agent-loop.md): that entry is about managing what accumulates *within* one loop; this one is about deciding what's worth carrying *between* loops in the first place.

## Not everything is worth persisting

The instinct once a memory mechanism exists is to save more — and that instinct produces the opposite of what memory is for. A recall system cluttered with facts that were only ever relevant once, or that are trivially re-derivable by looking at current state, surfaces stale and low-value entries with exactly as much apparent confidence as the few that actually matter. The useful filter isn't "could this be saved" but "would re-deriving this from scratch next time be more expensive than the cost of it going stale unnoticed."

**Not worth saving:** anything recoverable by reading current state — code structure, current dependencies, what a file currently does, recent git history. These are cheap to re-check and expensive to get wrong if a memorized snapshot silently diverges from reality. `git log` is authoritative; a memory *about* what git log said last month is not.

**Worth saving:** things that aren't written down anywhere else and can't be recovered by inspection — a preference stated explicitly with a reason, a decision whose rationale isn't visible in the artifact it produced, a standing fact about who's doing what and why. The test is whether the information would otherwise have to be re-explained by a person, not whether it's technically possible to store.

## The reasoning is the valuable part, not the fact

"User prefers X" is a weaker memory than "user prefers X because Y happened" — not because the second is longer, but because the reasoning is what lets a future judgment call be made correctly in a situation the original statement didn't explicitly cover. A bare rule generalizes badly at the edges; a rule with its reasoning attached lets a later decision ask "does the reason still apply here" instead of blindly pattern-matching the original wording. This is the same principle as comment discipline in [coding-patterns/agent-legible-code.md](../coding-patterns/agent-legible-code.md): what's worth recording is the *why*, because the *what* is often either obvious or independently recoverable.

## A memory is a claim about the world at the time it was written

Treat anything checkable in a memory — a file path, a function name, a specific fact about running state — as a claim to verify before acting on it, not a guarantee. Code gets renamed, files move, a decision gets reversed. A memory system that's queried and trusted without re-verification degrades into exactly the same failure mode as a stale comment or a stale repo-instructions file: confidently wrong, and trusted by default because nothing marks it as possibly outdated. The fix isn't to distrust memory generally — it's to re-check the specific, falsifiable parts of a memory before a recommendation built on it reaches someone who's about to act on it.

## An index avoids paying full cost for everything ever stored

A memory store that grows over a long relationship can't have every entry loaded in full on every session — that's the same fixed-cost-vs-actual-need problem as loading every tool's full schema (see [tool-schema-design.md](tool-schema-design.md)) or an overlong repo-instructions file (see [tools-and-skills/repo-level-agent-instructions.md](../tools-and-skills/repo-level-agent-instructions.md)). The pattern that scales: keep a lightweight, always-loaded index — short one-line pointers — and load a specific entry's full detail only when something in the current task actually calls for it. The index costs a fixed, small amount regardless of how much has accumulated; the detail costs something only when it's relevant.

## Memory isn't the same thing as a plan

A plan — the steps needed to finish the task currently in front of the loop — is transient by nature; it should be discarded or superseded once the task's done, not folded into durable knowledge about the user or the project. Persisting a plan as if it were memory pollutes future recall with information that was only ever meant to matter for one task's duration, and it's a different kind of information from "what does this user generally want" — conflating the two makes both harder to use correctly later.

## Anti-patterns

- **Saving what's derivable.** A memory that duplicates what a quick look at current state would tell you is pure liability — it can only go stale, never get more accurate on its own.
- **Duplicate entries instead of updates.** Writing a new memory every time related information comes up, rather than checking whether an existing entry should be revised, produces near-duplicates that eventually disagree with each other with no signal for which one's current.
- **Facts without reasoning.** A rule with no "why" is a trap for the first situation that doesn't quite match its literal wording.
- **Write-only memory.** Saving liberally but never revisiting or pruning what's already there means the store only grows, and its signal-to-noise ratio only drops, indefinitely.
- **Treating a task list as memory.** Scoped, transient planning state doesn't belong in the same store as durable facts about a user or project — it has a different lifetime and a different purpose.

## Related

- [agent-loop.md](agent-loop.md) — the within-session counterpart to this entry's across-session concern; both are versions of the same question: what's worth keeping, and what's cheaper to discard and re-derive.
- [tools-and-skills/repo-level-agent-instructions.md](../tools-and-skills/repo-level-agent-instructions.md) — a repo-scoped, manually-curated form of the same idea: durable context an agent shouldn't have to rediscover, deliberately kept short for the same reason.
