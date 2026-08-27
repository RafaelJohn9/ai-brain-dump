# Evaluating a Prompt or Skill, Specifically

[agentic-ai/evaluation.md](../agentic-ai/evaluation.md) covers evaluation at the scale of a whole agent: proxy metrics vs. actual task success, golden sets, grading strength. A single prompt template or skill — the kind cataloged in [prompts/](../prompts/) — is a much narrower unit than a whole agent, which makes it more tractable to actually evaluate, and also easier to skip evaluating entirely, because a prompt that "reads well" feels finished the moment it's written, whether or not it's ever been checked against a real input.

## Why a prompt is more tractable to evaluate than a whole agent

A prompt template usually has one specific job — interpret ambiguous intent, critique a UI, simulate a user's reaction to a product idea. That narrowness is what makes a small, focused golden set actually achievable: the space of inputs the prompt needs to handle well, and what "handled well" means for each one, is much better defined than "is this entire agent working," which is why this is worth treating as its own practice rather than assuming the general evaluation entry already covers it in enough detail to act on.

## Building a small golden set for one prompt

Eight to fifteen real or realistic inputs the prompt is actually meant to handle is enough to move from "I think this works" to "here's what it does on known cases" — and the set is only useful if it includes more than the clean example that motivated writing the prompt. A prompt tested only against the case that inspired it will always look like it works, because that case is precisely the one the prompt's author was thinking about while writing it. The inputs worth deliberately including: at least one genuinely ambiguous case, and at least one case at the edge of what the prompt should versus shouldn't handle — the cases most likely to reveal that the prompt's wording was clearer to its author than its behavior actually is.

## Grading: rubric, not vibes

A prompt's output is usually open-ended text, not a single correct value, which makes rubric-based grading — the middle strength tier from [evaluation.md](../agentic-ai/evaluation.md) — the natural fit: a short checklist specific to that one prompt's actual job ("did it ask a clarifying question instead of guessing when the input was genuinely ambiguous," "did it stay within the stated scope rather than drifting into a related but different task"), evaluated against each golden-set case. A generic "does this response seem good" judgment doesn't transfer between graders or hold steady over time the way a specific checklist does.

## Regression-testing a prompt edit

The same discipline [evaluation.md](../agentic-ai/evaluation.md) describes for a whole agent applies at the scope of one prompt: when a template gets edited — a line added, a constraint reworded to fix one observed failure — the golden set gets re-run before the edit counts as done. A change aimed at one case can regress a different case that was previously handled correctly, and a read-through of the new wording won't reveal that; only running the set will.

## Where this repo specifically falls short right now

None of the eight prompt templates in [prompts/](../prompts/) currently ship with example inputs, expected behavior, or anything resembling a golden set — worth stating plainly rather than implying this is already handled somewhere. The fix doesn't need to be a full automated harness: a short "known cases" section alongside a prompt template, even just three or four inputs with a note on what a good response looks like for each, is enough to turn "this prompt should work" into something checkable the next time the template changes.

## Anti-pattern: grading a prompt by re-reading its own wording

"This instruction is clearly worded, so the prompt is good" tests whether the prompt is *comprehensible*, not whether it produces the intended *behavior* — those are different questions, and clarity of the instructions is not evidence about correctness of what actually comes out the other side when a real input goes in. The only way to answer the second question is to run the prompt against real inputs and check what happens.

## Related

- [agentic-ai/evaluation.md](../agentic-ai/evaluation.md) — the general version of every principle in this entry, at the scale of a whole agent rather than one prompt.
- [prompts/](../prompts/) — the templates this entry's stated gap applies to directly.
