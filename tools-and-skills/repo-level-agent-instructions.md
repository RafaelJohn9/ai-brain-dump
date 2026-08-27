# Repo-Level Agent Instructions (CLAUDE.md / AGENTS.md)

A file — `CLAUDE.md` for Claude Code, converging with other tools on the shared `AGENTS.md` convention — that an agentic coding tool reads automatically at the start of a session, before it's touched a single file. It's the mechanism referenced but not detailed in [coding-patterns/agent-legible-code.md](../coding-patterns/agent-legible-code.md): a way to give an agent durable, repo-specific context up front instead of leaving it to rediscover, or guess, the same things every single session.

**See it applied:** this repo's own [`/CLAUDE.md`](../CLAUDE.md) — short, states the conventions actually used to build this knowledge base (how to add an entry, the no-filler bar, the visual-artifact and git conventions), and nothing more than that.

## The problem it solves

Without it, an agent facing a repo it hasn't worked in before has two options: ask (which interrupts flow for anything it could plausibly have been told up front) or guess (which is fast but inconsistent — a different session might infer a different convention from the same ambiguous code). Neither is free. Asking repeatedly for things a maintainer already knows the answer to wastes the person's time; guessing wrong produces a change that has to be caught in review and redone, after the fact, instead of never being made in the first place.

A repo-level instructions file trades a one-time cost (writing it, keeping it current) for eliminating that repeated cost — every session reads it once at the start instead of every session separately rediscovering or mis-guessing the same facts.

## What belongs in it

- **Commands that aren't reliably discoverable another way** — the actual test command if it's not just `npm test`, how to run a single test file, a build step that has to happen before tests will pass. Things an agent could technically figure out by reading `package.json` or a `Makefile`, but that are worth stating directly if there's any ambiguity (multiple test configs, a wrapper script that does more than the obvious command).
- **House conventions that aren't inferable from the code alone** — a preference the team holds that isn't visible by reading any single file (e.g., "always create new commits instead of amending," "never use `any` even where TypeScript would allow it").
- **Known traps** — a command that looks safe but isn't in this repo's context, a directory that looks like generated output but is actually hand-maintained, a dependency that can't be upgraded for a specific reason. These are exactly the things a guess gets wrong, because nothing about the code itself signals the danger.
- **Nonstandard structure** — where things live, when the layout doesn't match what a framework's defaults would suggest.
- **Explicit scope boundaries** — what the agent shouldn't touch without asking first (a deploy config, a generated file, a directory owned by another team).

## What doesn't belong in it

- **Anything obvious from reading the code.** If a maintainer would say "just look at the file, it's clear," restating it in the instructions file is redundant the moment it's written and wrong the moment the code changes without the doc being updated alongside it — the same staleness risk described for in-code comments in [agent-legible-code.md](../coding-patterns/agent-legible-code.md).
- **A full architecture document.** Link to one if it exists; don't duplicate it. A repo-level instructions file is read in full, unconditionally, at the start of every session — content only occasionally relevant belongs somewhere that's consulted on demand instead, not somewhere that costs context on every session regardless of the task.
- **Anything that changes often.** A fast-moving detail (current sprint priorities, a temporary workaround for a bug that'll be fixed next week) belongs in a ticket or a memory system with a timestamp, not in a file implicitly presented as durable ground truth. A stale instruction is worse than no instruction, because it's trusted by default rather than questioned.

## Keep it short — the cost is recurring, not one-time

Every line in the file is read on every session, whether or not that session's task touches the part of the repo the line is about. A ten-line file that states the handful of things actually worth knowing up front costs less, every single time, than a two-hundred-line file where most sessions only need three of those lines. This is the same tradeoff as tool-schema bloat in [tool-schema-design.md](../agentic-ai/tool-schema-design.md): a fixed cost that scales with what exists, not with what a given task actually needs, deserves active pruning.

## Scoping it in a monorepo

A single root-level file forces every session to carry every package's conventions into context, even when the task only touches one. Nested instruction files — a root file with the conventions that really are repo-wide, plus subdirectory files that add specificity for one package or service — let a session working in one area load only what's relevant to that area, with the root file as shared baseline.

## The AGENTS.md convergence

Multiple agentic coding tools have been converging on `AGENTS.md` as a shared filename specifically so one file can serve more than one tool, instead of every tool requiring its own separately-maintained variant of the same content. Where a project's actual tooling supports it, that convergence is worth taking — maintaining two files with the same intent that can silently drift apart is strictly worse than maintaining one. Where a specific tool still expects its own filename, the practical move is a stub in that filename pointing at the shared one, rather than a full duplicate.

## Maintenance

Treat changes to this file like changes to code, not like documentation cleanup deferred to "later": a PR that changes a convention (a new test command, a renamed directory, a newly-forbidden pattern) should update the instructions file in the *same* diff. An instructions file that's updated separately, after the fact, spends most of its life out of sync with the repo it's describing — and an agent reading it has no way to know which lines are still accurate and which are leftover from before the change.

## Related

- [`/CLAUDE.md`](../CLAUDE.md) — this repo's own instructions file, a live example of the "keep it short" principle above in practice.
- [coding-patterns/agent-legible-code.md](../coding-patterns/agent-legible-code.md) — where this file was first introduced, as the source of team convention an agent reads instead of inferring style from nearby files.
- [claude-code-skills-and-subagents.md](claude-code-skills-and-subagents.md) — a skill packages a *procedure* an agent should follow when a specific task comes up; this file packages *standing facts* about the repo an agent should know regardless of task. They compose: a skill's instructions can assume the repo-level context has already been read.
