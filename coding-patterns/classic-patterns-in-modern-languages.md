# Classic Design Patterns in Modern Languages

The Gang of Four patterns were written for languages without first-class functions, without built-in iteration protocols, without generics as ubiquitous as they are now. A real share of those patterns exist specifically to work around a limitation the language had at the time — which means applying them by rote in a language that no longer has that limitation reproduces the workaround's ceremony without the reason the workaround existed. The useful move isn't "patterns are outdated" or "patterns are timeless" — it's asking, for each one, whether it's compensating for something the language doesn't provide, or answering a structural question about how pieces should relate that no language feature resolves on its own.

## Patterns a language feature now provides directly

**Strategy** — a family of interchangeable classes implementing one shared interface, swapped at runtime — exists to work around the absence of functions as values. In a language where functions *are* values, the pattern collapses to a parameter that accepts a function.

```ts
// Strategy-pattern shape: a class hierarchy for something that varies
interface DiscountStrategy { apply(price: number): number; }
class PercentOff implements DiscountStrategy {
  constructor(private pct: number) {}
  apply(price: number) { return price * (1 - this.pct / 100); }
}
class FlatOff implements DiscountStrategy {
  constructor(private amount: number) {}
  apply(price: number) { return price - this.amount; }
}
function checkout(price: number, strategy: DiscountStrategy) { return strategy.apply(price); }

// The same idea, once functions are values:
function checkout(price: number, discount: (p: number) => number) { return discount(price); }
checkout(100, p => p * 0.9);          // percent off
checkout(100, p => p - 15);           // flat off
```

Nothing is lost in the second version — the interface, the two classes, and the constructors were all compensating for the absence of a plain function parameter.

**Iterator** — a class exposing `hasNext()`/`next()` — is now usually just the language's own iteration protocol (generators, `for...of`, `yield`). Hand-rolling it re-implements something the language already provides, tested and understood by everyone who already knows the language.

**Command** — wrapping an action as an object so it can be queued, logged, or undone — often collapses to storing a closure plus its arguments, in a language where closures capture their environment. The object wrapper was standing in for "a function plus the data it needs," which a closure already is.

**Observer**'s core idea — something changes, subscribers get notified — is real and durable, but hand-rolling the subscription machinery is usually the wrong call today: event emitters, reactive streams, or a framework's own state-subscription system are the same idea, already built, tested, and familiar to anyone joining the codebase. The pattern's *concept* survived; its *typical implementation* is now something to reach for off the shelf rather than write by hand.

## Patterns still genuinely earning their place

**Factory** — encapsulating "how do I correctly build this" when construction involves real logic or validation — is unrelated to first-class functions or iteration protocols; it answers a structural question (should callers know the construction details, or should that knowledge live in one place) that's independent of language features.

**Adapter** — translating one interface into another that existing code already expects — is the literal mechanism [architecture-patterns/layering-and-hexagonal.md](../architecture-patterns/layering-and-hexagonal.md) is built on: an adapter implements a port the domain defined, in the shape a specific piece of infrastructure needs. This isn't a workaround for a missing language feature; it's a real structural boundary.

**Decorator** — wrapping something to add behavior while keeping its original interface (logging, caching, retries layered around a core implementation) — is still a genuinely useful way to compose cross-cutting behavior, even though a modern language often expresses it as a function-wrapping-a-function rather than a class hierarchy.

## The filter

For any pattern under consideration: is it compensating for something this specific language doesn't have (in which case the language's native feature is very likely the better choice now), or is it answering a real question about how two pieces of the system should relate to each other (in which case it's probably still earning its place, whatever form its implementation takes)? Reaching for the classic class-hierarchy implementation of a pattern in a language with a simpler native way to express the same idea is the concrete tell that it's being applied because "that's how the pattern is done" rather than because it's solving the problem actually in front of you.

## Anti-pattern: ceremony because it's the named pattern

Implementing Strategy as an interface and a class per behavior in a language with first-class functions, purely because Strategy is the recognized name for "behavior that varies," is the same instinct [functional-patterns.md](functional-patterns.md)'s "where it's ceremony" section names at the level of a whole paradigm choice — here it shows up one level down, at the level of picking a specific pattern's implementation over the language's simpler native equivalent.

## Related

- [functional-patterns.md](functional-patterns.md) — first-class functions are the specific language feature that collapses several of the patterns named above.
- [architecture-patterns/layering-and-hexagonal.md](../architecture-patterns/layering-and-hexagonal.md) — Adapter as a load-bearing pattern, not a workaround, with its own dedicated treatment.
- [agent-legible-code.md](agent-legible-code.md) — no speculative abstraction, the broader instinct this entry applies specifically to inherited pattern vocabulary.
