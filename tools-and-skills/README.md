# Tools & Skills

A working catalog of the tools, CLIs, and reusable skills worth knowing in an agentic-coding workflow — what each one is for, and when to reach for it over the alternative.

**Status: growing.** See the [root ROADMAP](../ROADMAP.md) for what's planned.

## Contents

- [claude-code-skills-and-subagents.md](claude-code-skills-and-subagents.md) — how Skills and Subagents differ in Claude Code, when to reach for each, and a worked example from this repo's own restructure.
- [repo-level-agent-instructions.md](repo-level-agent-instructions.md) — CLAUDE.md / AGENTS.md: what belongs in it vs. what doesn't, why it should stay short, scoping it in a monorepo, and treating it like code that goes stale if not updated in the same diff as the change it describes.
- [claude-code-hooks.md](claude-code-hooks.md) — hooks as deterministic enforcement vs. instructions as guidance an agent is merely expected to follow, common uses (blocking dangerous actions, mechanical style enforcement, audit logging), the trust model, and pitfalls (over-broad firing, duplicating what CLAUDE.md already says).
- [mcp-server-integration.md](mcp-server-integration.md) — MCP as a shared integration protocol, deferred tool schemas, the trust boundary MCP introduces (external content isn't instructions), namespacing to prevent tool collisions, and treating a server connection as a capability grant.
- [evaluating-prompts-and-skills.md](evaluating-prompts-and-skills.md) — the scaled-down, practical version of agentic-ai/evaluation.md for one prompt template or skill: a small golden set, rubric grading, regression-testing an edit, and an honest note that none of this repo's own prompts have one yet.

## Scope

- **AI coding tools** — Claude Code and comparable agentic CLIs/IDEs: what they're good at, their failure modes, how to configure them per-project.
- **Agent skills** — reusable, packaged instructions for a recurring task (the kind this repo's own `.claude/skills` pattern demonstrates) — how to write one, when a skill beats a raw prompt.
- **Dev tooling** — linters, formatters, type checkers, and how they change once an AI agent is the one writing most of the code.
- **MCP servers & integrations** — connecting agents to external systems (browsers, APIs, databases) safely.
- **Evaluation tooling** — how to test that a tool/skill/prompt actually does what it claims.

## How entries should be written

One tool per entry: what it does, what it doesn't, and a real example of using it — not a marketing summary.
