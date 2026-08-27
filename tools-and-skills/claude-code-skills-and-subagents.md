# Claude Code: Skills vs. Subagents

Claude Code has two distinct mechanisms for extending what an agentic session can do beyond "one model, one context window, answering inline." They look similar from the outside (both package up a task and hand it off) but solve different problems. Confusing them is the most common mistake in setting either one up.

## Skills — packaged instructions, same session

A **skill** is a set of instructions for handling a recurring kind of task: a deploy checklist, a code-review rubric, a repo-specific workflow. It lives as a `SKILL.md` file (project-scoped under `.claude/skills/<name>/`, or user-scoped) with YAML frontmatter — a `name` and a one-line `description` — followed by the instructions themselves.

Two ways a skill gets triggered:

- **Explicitly**, when the user types `/<skill-name>` (a slash command).
- **Implicitly**, when the task at hand matches the skill's `description` closely enough that the agent invokes it before falling back to default behavior. This is why the description matters more than the body — it's the only part evaluated to decide *whether* to load the skill, so it needs concrete trigger language ("Use when the user asks to deploy X"), not a vague summary.

Once invoked, a skill's instructions load **into the current turn** and are followed in place of default behavior. Most skills run this way — inline, synchronous, same context. Some skills instead run in a subagent and hand back only the agent's name, with the real result arriving later as a notification; that's the exception, used when the skill's work is heavy enough to isolate.

**Use a skill when:** the task is a known, repeatable shape someone has already thought through — you want consistent behavior every time it comes up, not a one-off judgment call.

## Subagents — a task run by a separate agent instance

The **Agent tool** spawns a different kind of unit: a whole separate agent instance, with its own tool loop, that runs a task and reports back. Two flavors matter:

- **Fresh agents** (`general-purpose`, `Explore`, `Plan`, or a named type like `code-reviewer`) start with **zero context**. They don't know what you've been discussing, what you've tried, or why the task matters. The prompt you hand them has to be self-contained — file paths, what's already been ruled out, what "done" looks like. A terse command-style prompt to a fresh agent produces shallow, generic work; it has nothing else to draw on.
- **Forks** (`subagent_type: "fork"`) are different: a fork inherits your *entire* conversation so far and always runs on your model. You don't re-explain anything — the fork prompt is a directive ("go find X"), not a briefing.

The other axis that matters: subagents run in the **background** and keep their tool output **out of your context**. You get a completion notification later, not a live stream of everything they did. This is the actual reason to reach for one — not "this task is hard," but "the intermediate output (search results, file dumps, exploratory reads) isn't worth carrying in my context once I have the answer."

**Use a subagent when:** you need to do something exploratory or parallelizable and don't want the raw process cluttering the main conversation — not as a way to make a task feel more thorough. Spawning subagents for work you could just do inline wastes the user's time waiting on a cold start for no benefit.

## The distinction that actually matters

| | Skill | Subagent |
|---|---|---|
| What it packages | Instructions (a *how*) | An independent task execution (a *who*) |
| Context | Loads into current turn (usually) | Runs separately; only the result returns |
| Setup | Author writes it once, reused every time the trigger matches | Chosen per-task, on the fly |
| Right for | A known, repeatable workflow | Open-ended research or context-heavy exploration you don't want to keep |

A skill can itself *decide* to spawn a subagent as part of its instructions — the two compose. The question to ask when reaching for either: am I encoding a repeatable procedure (skill), or am I trying to keep this session's context clean while something exploratory happens (subagent)?

## Worked example: restructuring this repo

Building out this repo's [restructure](../ROADMAP.md) is a case where the subagent question actually came up. Surveying every file's purpose (reading eight prompt templates, three overlapping research drafts, a models README) generates a lot of "read this, note the gist" output that's only useful once, to make a decision — not something worth keeping verbatim in context afterward.

The concrete tradeoff: forking that survey out would have kept the raw file contents out of context, at the cost of a round trip and losing the ability to make small judgment calls (e.g., "these three research drafts are clearly drafts of the same doc, not distinct topics") without re-explaining what I'd just read. For a one-shot survey of a repo this size, reading inline and discarding the detail once the plan was made cost less than the round trip would have. The same call on a larger repo, or one where the survey needed to run alongside other work, would favor a fork.

That's the actual test, not a rule of thumb about size: **would I need this output again, and is losing live judgment calls on it worth not paying for it in context?**

## Anti-patterns

- **Spawning a subagent per trivial lookup.** If the answer is one grep away, doing it inline is faster than a cold-start round trip.
- **Writing a fresh-agent prompt like a fork prompt.** "Based on what we found, fix it" means nothing to an agent with no memory of "what we found." Fresh agents need the finding restated, not referenced.
- **A skill description that describes the skill instead of the trigger.** "Handles deployment tasks" doesn't tell the matcher when to fire. "Use when the user asks to deploy to staging or production" does.
- **Using a skill for a one-off.** If it won't recur, it's not worth authoring — just do the task.
