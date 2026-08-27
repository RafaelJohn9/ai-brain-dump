# Testing Patterns: What to Fake and What to Exercise

A test exists to make a claim about behavior checkable without a human re-reading and re-reasoning about the code every time it might have changed. The design question that determines whether a test suite is actually worth what it costs isn't "how do I mock this" — it's *what to fake and what to exercise for real*. Get it wrong in one direction and tests pass while real bugs slip through; get it wrong in the other and tests are too slow or too brittle to actually run often enough to matter.

## Three kinds of test double, not one

"Mocking" gets used as a catch-all, but the three common test doubles serve different purposes:

- **Stub** — returns canned data when called; exists to control what a test's inputs are, not to check anything about how it was called.
- **Mock** — records calls and lets the test assert on them ("was this called exactly once, with these arguments"); exists to verify *interaction*, not data flow.
- **Fake** — a lightweight but real implementation (an in-memory database standing in for a production one); exists to get realistic behavior without the cost or dependency of the real thing.

Reaching for a mock when a stub would do adds an assertion about *how* a function was called on top of what it returned — which makes the test fail on legitimate refactors that change the call pattern without changing the actual behavior. That's often the real source of "our tests are brittle," not that mocking itself is bad.

## The boundary heuristic

The same framing from [error-handling-boundaries.md](error-handling-boundaries.md) applies here: fake or stub things at the boundary of what the test doesn't own or can't afford to run for real — the network, wall-clock time, an external API, a payment provider, filesystem writes in a fast unit test. Exercise your own logic for real, because that's specifically the part a test is supposed to be checking. A test that fakes its own logic and only exercises the boundary has inverted the responsibility — it's now testing that the fakes behave the way the fakes were told to behave, not that the code does the right thing.

This is a well-known, commonly cited real failure pattern: an integration test that mocks the database layer can pass every time, including the day a schema migration silently breaks production, because the thing that would have caught it — the real database actually applying the real migration — was exactly what got faked away. The mock wasn't wrong to exist; it was drawn around the wrong boundary, faking the one dependency the test most needed to be real to be worth anything.

## Before / after: testing the fake instead of the logic

**Before** — mocks every collaborator, including the one thing worth checking:

```python
def test_apply_discount():
    pricing_service = Mock()
    pricing_service.calculate.return_value = 90  # hand the test its own answer

    result = apply_discount(pricing_service, base_price=100, discount_pct=10)

    assert result == 90
    pricing_service.calculate.assert_called_once()
```

This test passes regardless of whether `apply_discount`'s actual discount math is correct — `calculate` was told what to return, so the assertion just confirms the mock did what it was configured to do. If the real discount logic divided by the wrong number, this test wouldn't notice.

**After** — the real logic runs; only the true external boundary (say, a currency-conversion API call inside the pricing service) is faked:

```python
def test_apply_discount():
    pricing_service = PricingService(currency_api=FakeCurrencyApi(rate=1.0))

    result = apply_discount(pricing_service, base_price=100, discount_pct=10)

    assert result == 90  # now actually exercises the discount math
```

`FakeCurrencyApi` replaces the one genuinely external, non-deterministic dependency (a network call to a rate provider); everything else — the discount calculation itself — runs as real code, which is what makes the assertion mean something.

## Property-based testing, for the cases example-based tests miss

Hand-picked example inputs test the cases someone thought to write down — which tend to be the cases that were already top of mind, and miss the ones that weren't. Property-based testing inverts this: state an invariant that should hold for an entire class of inputs (`decode(encode(x)) == x` for any `x`; a sort function's output is always the same length as its input) and let the framework generate a wide range of inputs, shrinking any failure down to the smallest case that still reproduces it. This is a strong fit for parsers, serialization round-trips, and anything else with a clean, statable invariant — and a poor fit for behavior that's inherently about specific, named scenarios rather than a general rule ("a new user's welcome email contains their name" isn't a property, it's an example).

## Fixtures: shared setup that shouldn't become shared coupling

A fixture — reusable setup shared across several tests — saves real duplication when scoped to what a specific group of tests actually needs. The trap is a fixture that grows over time to cover more and more tests' needs until it's effectively global: at that point, changing it to accommodate one test's new requirement risks silently breaking assumptions in every other test that happens to use it, even though those tests look unrelated to the change. A narrowly scoped fixture, reused only by the tests that genuinely share the same setup, keeps that coupling from forming in the first place.

## Why this connects back to the agent loop

A test suite is what makes "self-verification as part of the change" — the pattern named in [agent-legible-code.md](agent-legible-code.md) — actually possible rather than aspirational. Without a check that can run and confirm behavior, "done" collapses to "I believe this works," which is precisely the premature-stopping failure mode described in [agentic-ai/agent-loop.md](../agentic-ai/agent-loop.md): a claim of success with nothing behind it but the model's own confidence. A well-scoped test — real logic exercised, only true boundaries faked — is what turns that claim into something checked.

## Related

- [error-handling-boundaries.md](error-handling-boundaries.md) — the same boundary-drawing logic (validate/fake at the edge, trust what's inside it) applied to error handling instead of test design.
- [agentic-ai/agent-loop.md](../agentic-ai/agent-loop.md) — premature stopping, and why a real verification step is what prevents it.
