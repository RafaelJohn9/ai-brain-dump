# Coding Patterns

Code-level patterns — smaller in scope than [architecture-patterns](../architecture-patterns/): how to write a function, module, or class well, not how to lay out a whole system.

**Status: growing.** See the [root ROADMAP](../ROADMAP.md) for what's planned.

## Contents

- [agent-legible-code.md](agent-legible-code.md) — structuring code so an AI coding agent can modify it safely: locality of change, explicit boundaries, self-verification, comment discipline, and why speculative abstraction is riskier under agent-driven changes than human ones.
- [error-handling-boundaries.md](error-handling-boundaries.md) — exceptions vs. result types vs. error codes, why the mechanism matters less than *where* handling happens, with a before/after refactor moving validation to a single boundary.
- [testing-patterns.md](testing-patterns.md) — mocks vs. stubs vs. fakes, the boundary heuristic for what to fake vs. exercise for real, a before/after example of a test that only checks its own mocks, property-based testing, and fixture scope creep.
- [functional-patterns.md](functional-patterns.md) — pure functions, immutability via structural sharing (not naive copying), composition over one large function, a before/after example separating computation from a hidden side effect, and where the discipline pays off vs. adds ceremony.

## Scope

- **Classic design patterns** — the useful subset of GoF patterns in modern languages, and which ones are usually unneeded now (e.g. superseded by first-class functions).
- **Functional patterns** — pure functions, composition, immutability, where they pay off vs. add ceremony.
- **Error handling** — result types vs. exceptions vs. error codes, and picking one per boundary.
- **Testing patterns** — test doubles, fixtures, property-based testing, what to mock and what not to.
- **LLM-assisted coding patterns** — structuring code so an AI pair-programmer (or agent) can safely modify it: small diffs, clear boundaries, self-checking tests.
- **Anti-patterns** — named smells worth calling out explicitly (premature abstraction, God objects, shotgun surgery).

## How entries should be written

Show the before/after in real code, not pseudocode. State the specific failure the pattern prevents.
