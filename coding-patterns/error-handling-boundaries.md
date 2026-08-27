# Error Handling at Boundaries

Error handling scattered without a strategy tends to produce two failure modes at the same time, in different parts of the same codebase: errors that get silently swallowed somewhere and vanish, and defensive checks duplicated at every layer for conditions that were already ruled out one layer up. Both come from the same missing decision — nobody decided *where* an error should actually be handled, so it gets handled everywhere a little and nowhere completely.

## The mechanisms

| Mechanism | Failure is | Caller can ignore it? |
|---|---|---|
| Exceptions | Thrown, propagates until caught | Yes, silently — nothing forces a `catch` |
| Result/Either types | A value: `Ok(x) \| Err(e)` | No — the success value is inaccessible without handling the error case first, in a language that enforces it |
| Error codes / sentinels | A special return value (`-1`, `null`, `NaN`) | Yes, easily — indistinguishable from a valid value if the check is skipped |

The mechanism matters less than most debates about it suggest. What matters is *where in the call chain* handling actually happens.

## The real decision: where's the boundary

The question isn't "exceptions or result types, globally" — it's "where is untrusted or unvalidated data entering this system, and does every function downstream of that point actually need to re-check it." A boundary is any point where data crosses from something the code doesn't control (user input, a network response, a file, an external API) into code that does. Validate and handle failure *there*. Once data is past that point, treat it as trusted — internal functions calling other internal functions shouldn't be re-validating invariants the boundary already established.

**Before** — validation smeared across every layer, because nobody decided who owns it:

```ts
function parseAge(input: string): number {
  if (!input) throw new Error("missing input");
  const n = Number(input);
  if (isNaN(n)) throw new Error("not a number");
  return n;
}

function validateUser(user: { age: number }): void {
  // re-checking something the caller should have already ensured
  if (user.age === undefined || isNaN(user.age)) {
    throw new Error("invalid age");
  }
  if (user.age < 0) throw new Error("age cannot be negative");
}

function createUser(rawAge: string): User {
  const age = parseAge(rawAge);
  const user = { age };
  validateUser(user); // re-validates what parseAge already guaranteed
  return { ...user, id: generateId() };
}
```

`validateUser` doesn't trust its own caller, even though `createUser` is the only caller and it just finished producing a value `parseAge` already validated. That defensive re-check isn't protecting against a real scenario — it's protecting against the possibility that someday, some other caller might call `validateUser` directly with bad data. If that's a real future need, it's a reason to make `validateUser` *the* boundary — not a reason to check the same thing twice on every call that already goes through `parseAge`.

**After** — one boundary, explicit in the type, everything past it trusted:

```ts
type Result<T, E> = { ok: true; value: T } | { ok: false; error: E };

function parseAge(input: string): Result<number, string> {
  if (!input) return { ok: false, error: "missing input" };
  const n = Number(input);
  if (isNaN(n)) return { ok: false, error: `expected a number, got "${input}"` };
  if (n < 0) return { ok: false, error: "age cannot be negative" };
  return { ok: true, value: n };
}

// Internal from here down: age is a plain number, no re-validation needed.
function createUser(age: number): User {
  return { id: generateId(), age };
}

// The boundary: the only place untrusted input is handled.
function handleSignup(rawAge: string): User | null {
  const result = parseAge(rawAge);
  if (!result.ok) {
    log(result.error);
    return null;
  }
  return createUser(result.value);
}
```

`createUser` takes a plain `number` and has nothing to check — its signature *is* the guarantee that validation already happened. `handleSignup` is the only place that deals with the possibility of bad input, because it's the only place bad input can actually arrive from.

## Why a result type forces the issue and an exception doesn't

In `parseAge`'s exception version, nothing stops a caller from simply not writing a `try`/`catch` — the code compiles, runs, and throws at a random point in production the first time bad input actually shows up. In the result-type version, `result.value` isn't reachable without going through the `ok` check first — in a language with a type checker that enforces this (TypeScript with strict narrowing, Rust's `Result`, Swift's enums), skipping the check is a compile error, not a runtime surprise. That's the actual argument for result types at a boundary: not that exceptions are bad, but that a type system can make "did you handle the failure case" a static fact instead of a hope.

Exceptions still earn their place for genuinely exceptional conditions — the kind where forcing every single caller up the stack to explicitly handle them would be pure noise (out-of-memory, a broken database connection during an otherwise-routine query). The distinction worth holding onto: an *expected* failure mode of a boundary (bad user input, a 404 from an API) is a value the caller needs to see and consciously decide what to do with; a truly *exceptional* failure is one almost no caller can meaningfully recover from locally, and propagating it up to a single top-level handler is correct.

## Error messages are the interface

The same principle from [tool schema design](../agentic-ai/tool-schema-design.md) applies here: an error that says *what* failed and what a valid value looks like (`expected a number, got "twelve"`) lets whoever's handling it — a human debugging, or an agent reading the failure and deciding what to try next — fix the actual problem. An error that just says `"invalid input"` or returns a bare error code forces a second lookup just to find out what was wrong in the first place.

## Anti-patterns

- **Catch-and-swallow** — `catch (e) {}` or `catch (e) { return null; }` with nothing logged. The failure disappears; the caller's assumption ("this returned normally, so it worked") becomes silently false.
- **Catch-log-rethrow-with-lost-context** — re-throwing a new generic error instead of wrapping the original, discarding the actual cause right before it would have been useful.
- **Re-validating the same thing at every layer** — the pattern in the "before" example above: each layer's defensive check isn't a safety net, it's a sign nobody decided which layer actually owns the check.

## Related

- [agent-legible-code.md](agent-legible-code.md) — "validate at boundaries, trust internally" as a general code-structure principle; this entry is that principle worked through in depth.
- [agentic-ai/tool-schema-design.md](../agentic-ai/tool-schema-design.md) — the same "an error should say what a valid input looks like" argument, applied to tool calls instead of function calls.
