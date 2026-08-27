# Layering and Hexagonal Architecture

Layering is about controlling the *direction* dependencies point, not about which folder a file lives in. A codebase can have `presentation/`, `business/`, and `data/` directories and still have no real layering at all, if code in `business/` freely imports a specific ORM model or a specific HTTP framework type. The folder structure is a hint about intent; the dependency graph is the actual architecture.

## Classic n-tier layering

The traditional shape: presentation → business logic → data access, with dependencies flowing one direction, top to bottom. Presentation code can call business logic; business logic can call data access; data access doesn't call back up. This is a real, useful discipline on its own — the common failure isn't the shape, it's that nothing actually *enforces* the direction. A business-logic function that imports a specific database row type has quietly become data-access code wearing a business-logic label, and nothing about the folder it's sitting in will catch that.

## Hexagonal (ports & adapters): inverting the direction

Hexagonal architecture takes the same underlying goal — keep the core logic from depending on infrastructure — and inverts the typical arrow. The domain (the actual business logic) sits at the center with **zero outward dependencies**. It defines **ports**: interfaces describing what it needs, in its own vocabulary ("a place to save an order," not "a Postgres connection"). **Adapters** sit outside the domain and implement those ports for a specific technology — a Postgres adapter, an in-memory adapter for tests, a REST adapter for delivering the domain's results to a client.

The direction that matters: the domain never imports an adapter. Adapters import the port interface the domain defined and implement it. Infrastructure depends on the domain — not the other way around, which is the reverse of how most codebases naturally grow, since it's usually easier to write "the code that saves an order" as a function that directly calls a database client than to first define what "saving an order" means independent of any database.

Clean Architecture and Onion Architecture are the same family under different names and different ring diagrams — dependencies point inward toward the domain in all three. Worth recognizing as one idea with three names rather than three competing architectures.

## What this actually buys

- **Testing.** A port can be satisfied by an in-memory adapter in tests and a real database adapter in production, with the domain logic itself completely unaware of which one is in play. This is the same boundary-drawing principle as [coding-patterns/testing-patterns.md](../coding-patterns/testing-patterns.md) — a port *is* the boundary where a fake gets substituted for something real, made explicit and structural instead of ad hoc.
- **Swappable infrastructure.** Moving from one database, message queue, or third-party API to another touches the adapter that implements the relevant port — the domain logic that doesn't care *how* an order gets saved, only *that* it can be, doesn't need to change at all.
- **Comprehensibility in isolation.** Someone reading the domain layer doesn't need to know which web framework or database is in use to understand what the business logic actually does — the ports describe intent in domain terms, not infrastructure terms.

## When it's not worth it

A small CRUD app, a short-lived script, or a prototype gains nothing from a ports-and-adapters boundary — the extra indirection (define an interface, implement it once, trace a call through an extra layer to find where the real work happens) is a cost paid immediately for a benefit (swappability, isolated testing) that may never actually get used. This is the same judgment call as [coding-patterns/agent-legible-code.md](../coding-patterns/agent-legible-code.md)'s "no speculative abstraction" — a port with exactly one implementation and no real plan for a second isn't a hexagonal architecture, it's an extra layer of indirection wearing the pattern's name.

The practical sequencing that avoids both failure modes: start with straightforward layered code where dependencies flow one direction by convention. Introduce an explicit port/adapter boundary specifically at the point where there's a real, current reason — you actually need to swap an implementation, or you actually need to isolate something for fast testing — not preemptively, everywhere, on day one.

## Where this sits relative to service boundaries

[service-boundaries.md](service-boundaries.md) is this same question — how much should depend on how much — asked at a different altitude. A hexagonal domain core can live entirely inside a single module of a modular monolith; the port/adapter boundary doesn't require a network call or a separate deployable unit to be worth having. Confusing the two — reaching for a network-separated service because a codebase needs better internal boundaries — pays a much higher cost (see that entry's list of what splitting into services actually costs) for a problem a module-level boundary would have solved.

## Anti-pattern: ceremony without a second implementation

Defining a port interface, writing exactly one adapter that implements it, and never planning a second, common though it is, provides none of hexagonal architecture's actual benefits — there's nothing being swapped, nothing being isolated for testing that couldn't be isolated more simply. It's worth naming because it's an easy trap: the pattern *feels* like the disciplined choice, which is precisely what makes it easy to apply reflexively instead of in response to an actual need.

## Related

- [coding-patterns/testing-patterns.md](../coding-patterns/testing-patterns.md) — the boundary-fake principle this pattern makes structural.
- [coding-patterns/agent-legible-code.md](../coding-patterns/agent-legible-code.md) — why an unused abstraction is a cost, not a safety margin.
- [service-boundaries.md](service-boundaries.md) — the same dependency-direction question asked at the level of whole services instead of a single module.
