# Designing the States Beyond the Happy Path

A screen designed only for its happy path — data loaded, the list has items, the action succeeded — is a screen designed for maybe half of what a user will actually encounter. The other states (nothing here yet, still loading, it failed, it partly worked) usually get improvised at the end if they get designed at all, and that's exactly why they're where the experience most often falls apart: not because the happy path was wrong, but because everything outside it was never really decided.

## Empty state: onboarding, not a dead end

For a lot of features, the empty state is the *first* thing a new user sees — before they've added anything, it's not a failure state, it's the actual starting point of using the feature. A blank list with the words "No items" wastes the one moment the interface has the user's full attention with nothing else competing for it.

A considered empty state does two things a bare one doesn't: it explains what *would* show up here once there's something to show, and it gives one specific, actionable next step — a button that does the thing, not a vague instruction to go do it somewhere else. "No projects yet — create your first one" with a button beats "No projects" with nothing underneath it, and the difference in effort to write it is close to zero.

## Loading state: skeletons and spinners aren't interchangeable

Two different tools get treated as one generic "loading" concept, and picking the wrong one throws away information the interface already has:

- **Skeleton screens** — gray silhouettes matching the eventual content's shape — work when the layout is known ahead of time and the wait is short. They reduce *perceived* wait because the eye has something structurally familiar to anticipate; the content doesn't pop in from nothing, it resolves from a shape that was already there.
- **Spinners** are honest about uncertainty but convey nothing about shape or expected duration. They're the right choice specifically when the content's eventual layout genuinely isn't known ahead of time, or the wait is short enough that a skeleton would just flash and add noise.

Using a spinner for a list whose shape is completely predictable (five rows of the same card, every time) discards information the interface already has about what's about to appear — a skeleton for that same list gives the user something concrete to anticipate for the same implementation cost.

## Optimistic UI needs a real rollback path

Updating the interface immediately when a user acts — before the server has actually confirmed anything — then reconciling or rolling back if the request fails, makes an interface feel instant instead of laggy. It only works honestly if the rollback path is real and visible: an optimistic update that silently fails, leaving the interface showing something that never actually happened with no indication anything went wrong, is worse than a slower, pessimistic update that waits for confirmation — because the user now believes a false thing about their own data, with nothing telling them otherwise.

## Error state: the UI-copy version of a boundary error message

[coding-patterns/error-handling-boundaries.md](../coding-patterns/error-handling-boundaries.md) argues that a good error says what went wrong and what a valid next step looks like. That principle doesn't stop at the backend — it's exactly the test for error-state copy. "Something went wrong" tells a user nothing they can act on. "Couldn't save — check your connection and try again" gives them one. A generic, undifferentiated error state showing up everywhere in an interface is usually a downstream symptom of the same problem described in that entry: if error handling happens too far from where the failure actually occurred, by the time it surfaces at the UI there's no specific information left to show — the boundary that should have captured *what* failed already discarded it.

## Partial/degraded state: show what worked

When a request partially succeeds — three of five items loaded, one integration timed out — the honest move is showing what actually did come back plus a specific note about what didn't, not silently hiding the failure (which looks like everything worked when it didn't) and not blocking the entire view over one partial failure (which throws away the parts that did succeed). Treating "not fully successful" as a binary with "totally failed" loses real information a user could otherwise act on.

## The actual discipline

For any view with dynamic content, explicitly design — or at minimum explicitly *name* — its zero, loading, error, and partial states before calling the view finished, not just the one state that happened to be on screen during a demo or a screenshot. This is the same self-review instinct as [avoiding-the-ai-generated-look.md](avoiding-the-ai-generated-look.md)'s accessibility note: polish concentrated in the state everyone looks at by default, with everything else left unconsidered, is a specific and common way a design falls short of what it looked like in review.

## Anti-patterns

- **An empty state that's just the happy-path layout with "no data" dropped in** — not actually designed, just whatever's left when the content is missing.
- **A spinner used where a skeleton would fit** — discarding known layout information for no benefit.
- **An optimistic update with no visible rollback** — the interface asserting something happened when it didn't, with nothing correcting the record.
- **One generic error message reused everywhere** — a sign the underlying error handling didn't preserve enough to be specific by the time it reached the screen.

## Related

- [error-handling-boundaries.md](../coding-patterns/error-handling-boundaries.md) — where the "an error should say what happened and what to do next" principle originates, at the code level this entry applies to UI copy.
- [avoiding-the-ai-generated-look.md](avoiding-the-ai-generated-look.md) — the broader pattern of a template's default state getting all the attention while its other states get none.
