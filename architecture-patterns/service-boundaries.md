# Service Boundaries: Monolith, Modular Monolith, Microservices

Choosing how many deployable units a system is split into — one, or many — before there's concrete evidence for what a specific split would fix, is one of the more expensive architecture decisions to get wrong. It's expensive specifically because of the asymmetry: staying a monolith too long costs coordination friction that's annoying but fixable; splitting into services too early costs an operational and consistency burden that's hard to undo, because undoing it means migrating running production systems back together, not just moving code within one codebase.

## The three shapes

**Monolith** — one deployable unit, one codebase, one process, one database. The right default for most projects, and especially before product-market fit: every unit of coordination overhead a split would introduce is overhead paid before there's evidence the product needs the flexibility it buys.

**Modular monolith** — still one deployable unit, but with internal boundaries enforced as if they were separate services: modules with explicit public interfaces, dependency-direction rules (module A can depend on B but not the reverse), sometimes an internal API layer between them even though it's all one process. This gets most of the organizational clarity people reach for microservices to get — clear ownership, limited blast radius for a change, the ability to reason about one module without loading the whole system into your head — without paying the operational cost of distribution. For a lot of systems, this is the right *permanent* architecture, not a stepping stone to something else.

**Microservices** — separate deployable units, typically separate data stores, communicating over the network. Right when there's a real, demonstrated need: a specific hot path that needs to scale independently of the rest of the system, teams that are genuinely blocked on each other's deploy schedules often enough that it's costing real time, or a real requirement to isolate a failure domain (a flaky third-party integration that shouldn't be able to take down checkout).

## What splitting into services actually costs

These costs are real regardless of how clean the resulting boundary is, which is why the decision to pay them needs a specific reason, not a general belief that microservices are the more mature architecture:

- **Function calls become network calls.** A call that used to be a few nanoseconds and couldn't fail becomes a request that can time out, get dropped, or arrive twice — a new category of failure that didn't exist before, and that every caller now has to handle.
- **Transactions become eventual consistency.** A single-database monolith gets atomic transactions for free. Once state is split across services, an operation that touches two of them needs an explicit strategy (sagas, outbox pattern, accepted eventual consistency) for what happens when the second half fails after the first half already committed — a problem that simply doesn't exist inside one database transaction.
- **Operational surface multiplies.** N services means N deploy pipelines, N sets of monitoring/alerting, N places a version-skew bug between "service A's new contract" and "service B's old client" can hide.
- **A wrong boundary is expensive to fix.** A module boundary drawn in the wrong place inside a monolith gets fixed by moving code and updating imports. A service boundary drawn in the wrong place gets fixed by migrating live data and coordinating a cutover across running production systems — the same mistake, an order of magnitude more expensive to correct.

## Where to actually draw the line

Draw boundaries around business capabilities — the things that change together because they represent one coherent piece of the domain (orders, billing, inventory) — not around technical layers (a "database service," an "auth service" carved out from everything that needs authentication just because auth sounds like it should be its own thing). A boundary drawn around a technical layer instead of a domain tends to need to change in lockstep with everything that calls it, which defeats the independence a service boundary is supposed to buy in the first place.

Conway's Law is worth taking seriously here in both directions: a service boundary that doesn't match how teams are actually organized creates coordination overhead no diagram can fix, and reorganizing the boundary to match team structure is often more effective than trying to reorganize the teams to match an idealized architecture.

## Knowing it's time to split

The trigger should be a specific, already-occurring pain, not a general sense that the codebase has gotten big: a module with a scaling profile clearly different from the rest of the system (needs 10x the compute, or needs to scale to zero when idle while everything else stays warm), two teams that can't ship independently because they keep colliding on the same deploy, or a compliance/isolation requirement that specifically demands separation. "It's gotten big" is a signal to consider a *modular monolith* refactor first — enforcing internal boundaries usually resolves the organizational pain that prompted the question, without yet paying the distribution cost.

## Anti-pattern: the distributed monolith

Services split along technical lines that still share a database, still have to deploy together because of tightly coupled contracts, and still can't be changed independently — this combines every cost of distribution (network calls, operational multiplication, harder debugging) with none of the benefit (independent scaling, independent deployment, isolated failure domains). It's usually the result of adopting microservices as a default rather than in response to a specific forcing function, and it's worth naming explicitly because it's common enough to have earned its own term.

## Related

- [coding-patterns/agent-legible-code.md](../coding-patterns/agent-legible-code.md) — "locality of change" is the same instinct as a good service boundary, applied at the level of a single diff instead of a whole system.
- [rag-pipeline.md](rag-pipeline.md) — an example of a natural boundary (offline indexing vs. online query) that's worth separating operationally even inside an otherwise monolithic system, because the two sides have genuinely different latency and scaling profiles.
