# Sagas

[service-boundaries.md](service-boundaries.md) names "transactions become eventual consistency" as one of the real costs of splitting into services: an operation that used to be one atomic database transaction, spanning tables in a single database, now spans multiple services with their own separate databases, and no single transaction can cover all of them at once. The saga pattern is the concrete answer to that specific problem — not a new idea on its own, but the standard shape for handling a multi-service operation once a plain distributed transaction isn't available.

## The shape

A saga is a sequence of local transactions, each scoped to one service, where each step's completion triggers the next step, and — critically — each step has a defined **compensating action** to undo it if a later step in the sequence fails. A booking flow might look like: charge the card → reserve the flight → reserve the hotel. If the hotel reservation fails, the saga runs compensating actions for the steps that already succeeded, in reverse: cancel the flight reservation, then refund the card. Nothing about this requires a single transaction spanning all three services — each step commits locally, and the saga's job is making sure a failure partway through doesn't leave the system in a state where some steps happened and others didn't with no accounting for the difference.

## Two ways to coordinate the sequence

- **Choreography** — each service listens for events from the others and decides its own next action independently, with no central coordinator. Simple for a small number of steps, and it avoids building a new coordinating component — but the overall flow isn't visible in any single place, which makes debugging and reasoning about the sequence harder as the number of steps grows past a handful.
- **Orchestration** — a dedicated saga orchestrator explicitly sequences each step and explicitly calls the compensating actions on failure. The flow is visible in one place, which is a real debugging and comprehension advantage, at the cost of a new component that has to be built, deployed, and kept correct.

This is the same choreography-vs-orchestration tradeoff named in [agentic-ai/multi-agent-orchestration.md](../agentic-ai/multi-agent-orchestration.md), applied to services coordinating a business transaction instead of agents coordinating a task — worth recognizing as the same underlying decision (does one party explicitly direct the others, or do all parties independently react to shared signals) rather than two unrelated concepts that happen to share vocabulary.

## Compensating actions aren't just "the reverse operation"

The hard part of a saga isn't the happy-path sequence — it's that undoing a step often isn't a clean inverse. A captured credit card charge can usually only be *refunded*, not made to have never happened; there's a real window where the charge existed and the system needs to be honest about it, not pretend the compensating action erased that window entirely. Anything downstream that observed "charged" as briefly true — a notification sent, a report generated — has to account for that, because the compensating action undoes the state, not the fact that it was briefly real. Sagas guarantee the system reaches a consistent end state; they don't guarantee the intermediate, partially-completed state never happened.

## When it's worth the pattern, and when it isn't

A saga is only needed once an operation genuinely spans multiple independently deployed services, each with its own database — an operation that stays inside a single service's single database still gets a real ACID transaction and doesn't need any of this. Reaching for saga machinery to coordinate steps that could just as easily live inside one service's transaction is paying this pattern's real complexity (defining every compensating action, building or adopting an orchestrator, reasoning about the partial-failure window) for a problem [service-boundaries.md](service-boundaries.md) would say shouldn't exist yet — the service split that created the need for a saga should itself have been driven by a real forcing function, not assumed as a starting point.

## Anti-pattern: a step with no compensating action

A saga step shipped without its compensating action defined — discovered only in production, at the exact moment a later step fails and that earlier step actually needs to be undone — is the pattern's most common and most expensive failure. Every step's undo path needs to be designed and tested before the saga ships, not written reactively after the first real failure demonstrates it was missing.

## Related

- [service-boundaries.md](service-boundaries.md) — the eventual-consistency cost this pattern exists to manage.
- [cqrs-and-event-sourcing.md](cqrs-and-event-sourcing.md) — a saga's steps are naturally expressed as events, and pair well with an event-sourced write side on either end of the sequence.
- [agentic-ai/multi-agent-orchestration.md](../agentic-ai/multi-agent-orchestration.md) — the same choreography-vs-orchestration coordination tradeoff, in a different domain.
