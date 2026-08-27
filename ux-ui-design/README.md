# UX / UI Design

Design knowledge for people building the interface, not just the backend — visual design fundamentals, interaction patterns, and how to work with AI design tools without shipping something generic.

**Status: growing.** See the [root ROADMAP](../ROADMAP.md) for what's planned.

## Contents

- [avoiding-the-ai-generated-look.md](avoiding-the-ai-generated-look.md) — the recognizable tells of generated-not-designed UI (default gradients, uniform rounded-shadow surfaces, decorative icons, template hero/feature-grid layouts), why they converge, what actually breaks the pattern, and a pre-ship self-review checklist.
- [design-tokens-and-theming.md](design-tokens-and-theming.md) — primitive vs. semantic tokens, the three-state (light/dark/system) theming problem and a CSS pattern that actually handles all three, and when to build a token system vs. adopt one.
- [color-scheme-patterns.md](color-scheme-patterns.md) — 12 color-scheme strategies (6 color-wheel relationships, 6 practical archetypes), each with a concrete token set, a when-to-use case, and a specific pitfall. Companion [visual specimen page](https://claude.ai/code/artifact/78b319f1-3d06-4937-947b-3495cf04ec3a) renders all 12 on the same demo card for direct comparison.
- [subject-driven-identity.md](subject-driven-identity.md) — the practice of deriving a palette, type pairing, and one structural motif from a subject's own world instead of a shared template. Companion [visual specimen page](https://claude.ai/code/artifact/0288efad-ff46-4f4d-a8b0-429c46174eae) shows 12 fully distinct identities (birthday, gallery, Halloween, gala, coffee, two Substack editions, winter, wedding, festival, market, bookshop).
- [interaction-states.md](interaction-states.md) — designing the states beyond the happy path: empty state as onboarding, skeletons vs. spinners, optimistic UI's rollback requirement, error copy as the UI-level version of a boundary error message, and partial/degraded states.

## Scope

- **Visual fundamentals** — typography, color systems, spacing/layout grids, hierarchy.
- **Interaction patterns** — navigation, forms, empty/loading/error states, feedback and affordance.
- **Design systems** — tokens, component libraries, when to build one vs. borrow one.
- **Accessibility** — contrast, keyboard nav, screen-reader support as a baseline, not an afterthought.
- **AI-assisted design** — prompting design tools well, reviewing AI-generated UI critically, avoiding the "obviously AI-generated" look.
- **Critique notes** — teardown of real interfaces: what works, what doesn't, why.

## Related

- [prompts/ui-ux-designer.md](../prompts/ui-ux-designer.md) is a ready-to-use system prompt for a UI/UX design assistant.
