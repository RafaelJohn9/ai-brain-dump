# CQRS and Event Sourcing

A single model — one set of tables, one shape — trying to serve both writes and reads forces a compromise on both sides. Writes need strict validation, normalization, and transactional integrity; reads often need a differently shaped, denormalized view that's fast to query without reconstructing it from a write-optimized structure on every request. A schema tuned for correct writes usually isn't the schema that would have been designed for fast reads, and vice versa.

## CQRS: separate the write model from the read model

Command Query Responsibility Segregation splits the model that handles writes (commands — "place this order," "update this profile") from the model that serves reads (queries — "show me this user's order history"). They can have entirely different shapes, live in different stores, and scale independently of each other. A system with far more reads than writes can scale its read side without touching the write side at all — the same "specific hot path that needs to scale independently" trigger from [service-boundaries.md](service-boundaries.md), now showing up inside a single domain's data model rather than between whole services.

The mechanism connecting the two sides: a command mutates the write model and emits something — an event, a change record — that propagates to update the read model. That propagation isn't instant. The read model is *eventually* consistent with the write model, which is the same eventual-consistency tradeoff [service-boundaries.md](service-boundaries.md) names as a real cost of splitting services, reappearing here at the data-model level: a read immediately after a write may not yet reflect it, and a system built on CQRS has to have an actual answer for what a user sees during that gap.

## Event sourcing: a related but separate idea

Event sourcing is often mentioned in the same breath as CQRS, and the two pair naturally, but they're not the same pattern. Instead of storing only current state, an event-sourced system stores the sequence of events that produced it, and derives current state by replaying that sequence. The benefit is a complete history for free — the system always knows *how* it arrived at the current state, not just what the current state is — and the ability to reconstruct state as of any past point, or build an entirely new read-model projection later by replaying the same history through new logic.

That's valuable specifically when the history itself is a requirement, not just a nice-to-have: financial ledgers, anything with a compliance or audit obligation, systems where "what happened and in what order" is a question users or auditors actually ask.

## What event sourcing costs

Replaying a long event history to reconstruct current state gets slow as the history grows, which means a real implementation needs periodic snapshotting to stay fast — an added piece of infrastructure with its own correctness requirements. Event schemas are harder to evolve than table schemas: an event written five years ago under an old shape still has to be replayable today, so changing what an event *means* requires versioning and migration strategies a simple table alteration doesn't. And debugging shifts in character — instead of asking "what's wrong with this row," the question becomes "which event in this sequence produced the wrong outcome," which is a different, often harder kind of investigation.

## When each is actually worth it

CQRS on its own is worth adopting when read and write patterns have genuinely diverged — different scale requirements, different shape requirements, or both. A smaller system where reads and writes are roughly symmetric in volume and shape gains little from separating them, and pays a real, ongoing cost for it: two models to keep in sync, eventual consistency to reason about on every screen that shows recently written data.

Event sourcing specifically is worth its cost when the history is a first-class requirement — not because it "sounds more robust" for a domain that has never once needed to answer "what happened before this," and only ever cares about current state.

## Anti-pattern: event sourcing without a read model

Adopting event sourcing and then reconstructing "current state" by replaying the full event history at read time, on every request, because no actual read-model projection was ever built, pays every one of the pattern's costs — replay complexity, harder schema evolution, harder debugging — without using the one benefit it exists to provide. If nothing in the system is actually querying the history for its own sake, and every read still just wants current state fast, the projection that CQRS would have built is the thing that was actually needed — the event log without it is ceremony wearing the pattern's name.

## Related

- [service-boundaries.md](service-boundaries.md) — the same eventual-consistency tradeoff and the same "split when a hot path genuinely needs to scale independently" trigger, at the level of whole services rather than one domain's data model.
- [rag-pipeline.md](rag-pipeline.md) — a different instance of the same underlying instinct: an offline/online split because writes (indexing) and reads (query-time retrieval) have genuinely different latency and access-pattern needs.
