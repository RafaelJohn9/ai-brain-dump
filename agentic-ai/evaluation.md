# Evaluating Agents: Plausible Isn't Correct

A language model is good at producing fluent, confident, plausible-sounding output almost independent of whether that output is actually right. That's exactly what makes evaluation necessary and easy to skip at the same time — the failure mode isn't an agent that visibly breaks, it's one that looks like it's working while quietly not, and "looks like it's working" is precisely the thing a model is best at producing whether or not it's true.

## Proxy metrics vs. actual task success

It's tempting to grade an agent on what's easy to check: did it respond, did the response look complete, did it avoid throwing an error, did the user not immediately complain. These are proxies — they correlate with success often enough to feel like a reasonable stand-in, but the correlation is exactly that: a correlation, not the thing itself, and it quietly breaks as a system changes. A prompt edit that makes responses sound more confident without making them more correct will improve every proxy metric above while making the system worse at its actual job. The only way to catch that is checking the actual job, not the signals that usually go along with it.

This is the same failure named as "premature stopping" in [agent-loop.md](agent-loop.md), scaled up from a single loop's self-assessment to an entire system's measurement of itself — a proxy metric is a system-level version of the model believing its own claim of success without verifying it.

## A golden set built from real cases, not just imagined ones

The practical foundation for evaluation is a fixed set of representative tasks with known-correct outcomes, run before shipping any change and re-run after. What makes a golden set actually useful is where its cases come from: a set built only from cases someone imagined ahead of time will systematically miss the cases production reveals as hard, because those are, almost by definition, the ones nobody thought to imagine. Every real failure surfaced after the fact is a candidate to add to the set — this is the direct analogue of [coding-patterns/testing-patterns.md](../coding-patterns/testing-patterns.md)'s test suite, applied to "did the agent's behavior actually work" instead of "does this function return the right value."

## Three ways to grade an output, in order of strength

- **Exact or structural match** — for tasks with a genuinely correct, checkable answer: did it call the right tool with the right arguments, does the output parse into the expected shape. Cheap and unambiguous where it applies, but only applies where "correct" is actually a single checkable thing.
- **Rubric-based grading** — for open-ended output with no single correct string: a checklist evaluated by a human or a separate model, scoring specific, named criteria ("did it cite a source," "did it flag the ambiguity instead of guessing") rather than an undifferentiated "is this good" judgment, which is too vague to be consistent between graders or over time.
- **Outcome-based checking** — the strongest signal: did the downstream effect actually happen correctly. Did the file change the way it was supposed to. Does the test the agent claimed passes actually pass when re-run independently, from a clean state, by something other than the agent's own say-so. Outcome checks can't be fooled by fluent-but-wrong output the way a "does this look plausible" check can, because they're checking the world, not the description of the world.

## Don't let the agent grade its own work uncritically

A model asked to assess its own output has the same blind spot that may have produced the mistake in the first place — if a wrong assumption shaped the original answer, that same assumption is available to shape the self-assessment of it, which is the evaluation-level version of the "context poisoning" failure mode from [agent-loop.md](agent-loop.md). Independent verification — a different model with no stake in the original answer, a deterministic check, a human — catches what asking the same reasoning to grade itself systematically can't.

## Evaluation is a suite that runs on every change, not a one-time gate

Treat the golden set the way a test suite gets treated: re-run on every meaningful change — a prompt edit, a tool schema revision, a model version bump — not just once before initial launch. What matters most isn't the raw pass rate at any one point but which specific cases flip from passing to failing (or the reverse) between runs; a case that regresses is a much more actionable signal than an aggregate score moving slightly.

## The overfitting trap

Once a golden set exists and people are actively optimizing against it, there's real pressure to tune behavior until those specific cases pass without the underlying capability actually generalizing — a version of Goodhart's Law: a measure that becomes a target stops being a good measure. The set itself becomes something to pass rather than something that reflects reality. The mitigation is structural, not willpower: keep the set growing with freshly discovered real cases rather than freezing it, and hold out some fraction of cases that aren't visible during the tuning process, so there's always an unseen check on whether improvement generalized or just fit the visible set.

## Anti-patterns

- **Eyeballing a few examples and shipping.** No golden set means no repeatable measurement — "it looked right when I tried it" isn't evaluation, it's a single anecdote treated as a result.
- **Grading presence over correctness.** Checking that an answer exists, is well-formatted, or doesn't error is a proxy metric wearing evaluation's clothes.
- **Self-grading with no independent check.** The agent's own transcript, assessed by the same agent, inherits whatever blind spot produced the original mistake.
- **A frozen eval suite.** One that never absorbs new failure cases discovered in production stops reflecting the system it's supposed to be measuring, and the pass rate it reports becomes decreasingly meaningful over time.

## Related

- [agent-loop.md](agent-loop.md) — premature stopping at the level of a single loop; this entry is the same failure considered at the level of measuring a whole system.
- [coding-patterns/testing-patterns.md](../coding-patterns/testing-patterns.md) — the same self-verification discipline, applied to code correctness rather than agent behavior.
- [agent-memory.md](agent-memory.md) — treating a memory as a checkable claim rather than a guarantee is the same epistemic habit this entry asks for at the level of an entire evaluation result.
