# Design Tokens and Theming

A design token is a named value — a color, a spacing unit, a font size — that components reference by name instead of by literal value. The point isn't the indirection for its own sake; it's that the *meaning* of a value (this is the accent color, this is the danger color) becomes something you can change in one place, instead of a value that's been copy-pasted into every component that happened to need "that blue" at the time it was written.

## Two layers, not one

Tokens work best split into two layers that reference each other:

- **Primitive tokens** — raw values with no meaning attached: `--blue-500: #3b82f6`, `--space-4: 16px`. These describe *what a value is*.
- **Semantic tokens** — named by role, pointing at a primitive: `--color-accent: var(--blue-500)`, `--surface-bg: var(--gray-50)`. These describe *what a value is for*.

Components should reference semantic tokens only, never primitives directly. The reason this matters is entirely about what happens when something changes: if a component references `--blue-500` and the accent color needs to become teal, every component that used blue-for-accent-reasons has to be found and edited individually — and so does every component that used blue-500 for an unrelated reason, which now has to be disentangled from the ones that need to change. If components reference `--color-accent`, retargeting that one semantic token to a different primitive changes every consumer at once, correctly, because the *meaning* was captured at the point of use, not the value.

## Theming has three states, not two

A theme toggle looks like a two-state problem (light, dark) but is actually three: explicit light, explicit dark, and "system" — where the page has no explicit preference and defers to the OS/browser setting. Most theming implementations handle two of these correctly and get the third wrong, usually by defining dark-mode values only inside a `prefers-color-scheme` media query, which then can't be overridden by an explicit in-app toggle without extra work that often doesn't get added until a bug report shows up.

A pattern that handles all three correctly with CSS custom properties:

```css
/* Base layer: the full light palette, unconditionally */
:root {
  --color-bg: #ffffff;
  --color-text: #111111;
  --color-accent: #3b82f6;
}

/* System preference, only when no explicit choice has been made */
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --color-bg: #111111;
    --color-text: #f5f5f5;
    --color-accent: #60a5fa;
  }
}

/* Explicit override, wins in both directions regardless of system preference */
:root[data-theme="dark"] {
  --color-bg: #111111;
  --color-text: #f5f5f5;
  --color-accent: #60a5fa;
}
```

The toggle then just sets `data-theme="dark"` or `data-theme="light"` on the root element (or removes it, to fall back to system). Two things make this actually work: the light palette lives on bare `:root` with no guard, so it's the real default rather than something that only exists by absence of the dark rules; and the dark values get defined twice — once behind the media query, once behind the explicit attribute — so an explicit choice always wins over system preference in both directions, and system preference still works when no explicit choice has been made.

## A color's only definition should never live inside a conditional block

If `--color-accent` is defined *only* inside the dark-mode media query, then in any context where that media query doesn't apply — a print stylesheet, an embedded/sandboxed view, a browser quirk — the token simply doesn't exist, and whatever references it falls back to nothing or inherits unpredictably. Every token needs a real definition in the unconditional base layer; conditional blocks should only ever *override* it.

## Give surfaces an explicit background token

A component with `background: transparent` (or no background declared at all) doesn't have "no color" — it shows whatever is behind it, which means its actual appearance depends on where it happens to get placed. Giving every surface-level component an explicit `background: var(--surface-bg)` token means it looks correct regardless of context, and means the surface color is itself themeable through the same token system instead of being an accident of nesting.

## Spacing and type scales are tokens too

The same reasoning that applies to color applies to spacing and typography: pick a scale once (a 4px or 8px base unit, a type ratio like 1.25×) and reference steps in it by name (`--space-2`, `--space-4`, `--text-lg`) rather than writing one-off pixel values per component. A one-off `margin: 13px` might be the exact right visual adjustment in isolation, but it's a value nothing else in the system knows about — it can't be part of a consistent rhythm because it isn't drawn from the same set anything else is drawn from.

## When to build a token system vs. adopt one

Building a full token system from scratch pays off when a product has enough surface area and lifetime to amortize the setup cost, and when an off-the-shelf visual identity (an unmodified component library's default look) would actually undermine what the product is trying to be. For most early-stage products and internal tools, adopting an existing component library and overriding its token layer gets most of the consistency benefit for a fraction of the cost — the mistake is either direction taken reflexively: building a bespoke system before there's enough product to need one, or accepting a library's raw defaults indefinitely because building a token layer felt like premature investment.

## Anti-patterns

- **Hardcoded values inside components** — a hex color or pixel value written directly into component styles is invisible to the token system and has to be found and fixed manually every time the design changes.
- **Tokens that are just renamed primitives** — `--color-blue` is not a semantic token; it describes the value, not the role, and provides none of the retargeting benefit a real semantic layer provides.
- **Two sources of truth** — token values duplicated separately in JS (for a component library's theme object) and CSS (for custom properties), which inevitably drift out of sync as one gets updated without the other.

## Related

- [avoiding-the-ai-generated-look.md](avoiding-the-ai-generated-look.md) — a constrained, intentional palette (what a real token system produces) is the concrete alternative to the decorative-gradient default described there.
