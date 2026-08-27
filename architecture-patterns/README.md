# Architecture Patterns

Structural patterns for organizing a project — how pieces are divided and how they're allowed to talk to each other. Framework-agnostic; each pattern should note where it fits and where it doesn't.

**Status: growing.** See the [root ROADMAP](../ROADMAP.md) for what's planned.

## Contents

- [rag-pipeline.md](rag-pipeline.md) — RAG pipeline architecture: offline/online pipeline split, component boundaries, chunking and retrieval tradeoffs, variants, and when not to use it.
- [service-boundaries.md](service-boundaries.md) — monolith vs. modular monolith vs. microservices: what splitting into services actually costs, where to draw the boundary, and the distributed-monolith anti-pattern.
- [layering-and-hexagonal.md](layering-and-hexagonal.md) — layering as dependency direction rather than folder names, ports & adapters as the inverted version, what it actually buys (testability, swappable infrastructure), and the ceremony-without-a-second-implementation anti-pattern.

## Scope

- **Layering** — layered/n-tier, hexagonal (ports & adapters), clean architecture.
- **Service boundaries** — monolith vs. modular monolith vs. microservices, and the tradeoffs at each size.
- **Data patterns** — event-driven architecture, CQRS, event sourcing.
- **Frontend architecture** — component composition strategies, state management boundaries, server/client split (SSR, islands, RSC-style patterns).
- **AI-system-specific patterns** — RAG pipeline architecture, agent orchestration topology, model-serving layers.
- **Case studies** — real project write-ups: what pattern was chosen, why, what it cost.

## How entries should be written

Every pattern entry should state the problem it solves, not just describe the shape. A diagram plus "use this when X, avoid it when Y" beats an abstract definition.
