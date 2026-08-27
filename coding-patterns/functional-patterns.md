# Functional Patterns: Pure Functions, Immutability, Composition

State mutated in place and shared across a call chain makes a program hard to reason about for a specific reason: understanding what a function does stops being about reading that function's body and becomes about knowing everything else that might have touched the shared data before it ran. Functional patterns are, underneath the terminology, a set of ways to remove that "everything else" dependency so a piece of code can be understood — and tested — on its own.

## Pure functions: the part of a system that's cheapest to trust

A pure function returns the same output for the same input and produces no observable side effect. The value isn't abstract — a pure function can be tested with nothing but its inputs and expected output, no setup or teardown of surrounding state required; it can be reordered, cached, or run concurrently without risk, because it doesn't depend on or affect anything outside itself; and it can be understood completely by reading its signature and body, without reconstructing the history of everything that ran before it.

This is precisely the code [coding-patterns/testing-patterns.md](testing-patterns.md) describes exercising for real rather than faking — a pure function has no external boundary hiding inside it that a test would need to fake in the first place. The more of a codebase's actual logic is pure, the more of it falls into the "exercise for real" category by construction.

## Immutability: a new value, not a changed one

Instead of mutating a data structure in place, an update produces a new structure with the change applied, leaving the original untouched. The direct benefit: anything else still holding a reference to the old value keeps seeing the old value — nothing gets silently surprised by a change it didn't ask for and doesn't know happened.

The naive version of this — copying an entire large structure on every small change — is genuinely expensive, and it's worth being precise that "immutable" doesn't have to mean that. Real immutable data structures typically use **structural sharing**: only the part of the structure that actually changed gets copied, and everything else is shared by reference between the old and new versions. Immutability's cost is closer to "the size of the change" than "the size of the whole structure," in a well-implemented version of the pattern.

## Composition over one large function

Building a capability by chaining small, single-purpose functions — `pipe(parse, validate, transform)` — instead of one large function that does all three inline, or a class hierarchy encoding the combination, means each piece can be tested, replaced, or reordered independently of the others. This is [coding-patterns/agent-legible-code.md](agent-legible-code.md)'s "locality of change" argument, one level down: a composed pipeline confines the blast radius of fixing the validation step to the validation function, rather than requiring a careful edit inside a much larger function that also parses and transforms.

## Before / after: separating computation from its side effect

**Before** — one function both computes a result and mutates shared state as a side effect:

```ts
let auditLog: string[] = [];

function applyDiscount(price: number, pct: number): number {
  const result = price * (1 - pct / 100);
  auditLog.push(`Applied ${pct}% discount to ${price} -> ${result}`); // hidden side effect
  return result;
}
```

Testing the discount math means also dealing with `auditLog` — asserting on it, resetting it between tests, or accepting that the test's real intent (is the math right?) is entangled with something unrelated (did the logging happen?). Every caller of `applyDiscount` is also, invisibly, a caller of the logging system, whether or not it wants to be.

**After** — the calculation is pure; recording it is a separate, explicit step:

```ts
function applyDiscount(price: number, pct: number): number {
  return price * (1 - pct / 100);
}

function logDiscount(entry: string, log: string[]): string[] {
  return [...log, entry]; // new array, original untouched
}

const result = applyDiscount(100, 10);
const auditLog = logDiscount(`Applied 10% discount to 100 -> ${result}`, auditLog);
```

`applyDiscount` can now be tested with nothing but numbers in and a number out. Whether or not to log, and what the log data structure looks like, is a decision made at the call site — visible, not buried inside the function that computes the discount.

## Where it pays off, and where it's ceremony

The discipline pays off most in logic that's reused, tested independently, or ever needs to run concurrently — the actual business rules of a system. It turns into ceremony when applied reflexively to code that's inherently about managing a single piece of local, short-lived mutable state close to where it's used — a UI component's own interaction state, a tight loop doing in-place array processing where allocation overhead is the whole performance concern. Forcing immutability onto code like that adds real cost (allocation, indirection) for a guarantee nothing else needed, because nothing else was ever going to hold a stale reference to that state in the first place.

This is the same judgment call as "no speculative abstraction" from [agent-legible-code.md](agent-legible-code.md), applied to a paradigm choice instead of a structural one: adopt the discipline where its actual benefit — testability in isolation, safety when a value is shared — is genuinely needed, not as a style rule applied uniformly regardless of whether anything is being shared.

## Anti-pattern: cosmetic purity

A function that looks pure on the outside but calls another function one layer down that mutates a module-level variable isn't actually pure — it's a pure-looking wrapper around hidden mutation. The discipline only delivers what it promises (testability without setup, safety under sharing) if it holds all the way through the call chain a function actually depends on, not just at its own top level.

## Related

- [testing-patterns.md](testing-patterns.md) — pure functions are exactly the kind of logic that entry says should be exercised for real rather than faked.
- [agent-legible-code.md](agent-legible-code.md) — locality of change and no-speculative-abstraction, the same instincts this entry applies to a functional-vs-imperative choice instead of a file-level one.
