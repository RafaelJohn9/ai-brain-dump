# Color Scheme Patterns

A color scheme *strategy* is a repeatable relationship between hues — not a specific palette. "Complementary" describes a relationship (two hues opposite each other on the wheel); the actual hex values you pick to embody it are the palette. This entry catalogs 12 strategies — 6 classic color-wheel relationships and 6 practical archetypes that show up repeatedly in real UI work — each with a concrete token set, so it's usable directly, not just theoretical.

**See them rendered:** [Color Scheme Specimens](https://claude.ai/code/artifact/78b319f1-3d06-4937-947b-3495cf04ec3a) — all 12 applied to the same demo card, so they're directly comparable rather than described in the abstract.

This is the "which relationship to pick" question that comes *before* [design-tokens-and-theming.md](design-tokens-and-theming.md)'s "how to encode the result" question. Pick a strategy first; turn its output into primitive and semantic tokens second.

## Color-Wheel Strategies

### 1. Monochromatic

One hue, carried across a range of lightness and saturation. Hierarchy comes entirely from value (how light or dark), not from switching hues.

- `bg` `#F0F5F4` · `text` `#0F2E2B` · `accent` `#0F766E` · `tag` `#99C7C1`
- **Use when:** the UI should feel calm and focused, and you want hierarchy to come from a value scale rather than color-coding.
- **Watch for:** it flattens easily on screen — lean hard on contrast in lightness, not just a slight tint shift, or elements stop reading as distinct.

### 2. Analogous

Three hues that sit next to each other on the wheel. Naturally harmonious — warm or cool without any hue actively opposing another.

- `bg` `#F3F1FA` · `text` `#241B4E` · `accent` `#5B3FA6` · `tag 2` `#CFE0F5`
- **Use when:** building an editorial or creative product that wants warmth and variety without a jarring clash.
- **Watch for:** with no true opposite in the palette, there's nothing to punch through for emphasis — add one sharp, unrelated neutral (near-black or near-white) to anchor contrast.

### 3. Complementary

Two hues directly opposite each other. The contrast between them does the hierarchy's job for you — nothing else needs to work as hard.

- `bg` `#FBF3EC` · `text` `#2B2013` · `accent` `#B0491F` · `tag 2` `#1E3A5F`
- **Use when:** a single primary action needs to be unmissable against a warm, otherwise-quiet ground.
- **Watch for:** at full saturation, true complements visually vibrate against each other — desaturate one side (usually the background hue) so the pairing reads as intentional rather than jarring.

### 4. Split-Complementary

A base hue plus both neighbors of its direct opposite, instead of the opposite itself. Same structural punch as complementary, gentler to look at over a long session.

- `bg` `#FBF7EC` · `text` `#2B2510` · `accent` `#5A4B8C` · `tag` `#C99A2E` · `tag 2` `#8C4B6B`
- **Use when:** you want complementary's contrast without complementary's intensity — dashboards or tools people stare at for hours.
- **Watch for:** three hues are now in active play — assign each a strict, single job (primary action, one tag family, one secondary accent) or the palette reads as noisy rather than considered.

### 5. Triadic

Three hues evenly spaced around the wheel, each pulling roughly equal visual weight.

- `bg` `#F7F4EF` · `text` `#241F1A` · `accent` `#B3462C` · `tag` `#C99A2E` · `ghost` `#33628C`
- **Use when:** the interface genuinely needs three co-equal categories to read at a glance — status, priority, and type, for instance — and no one of them should dominate.
- **Watch for:** balance saturation across all three deliberately, or whichever one happens to be most saturated will read as "the real accent" and the other two will feel like afterthoughts.

### 6. Tetradic (Square)

Two complementary pairs at once — about the most distinct hues a single UI can hold before it starts fighting itself.

- `bg` `#F5F4F0` · `accent` `#1F3A5F` · `tag` `#B98A2E` · `tag 2` `#6B3A5C` · `reserve` `#587A5C`
- **Use when:** a data-dense UI truly needs four distinct categorical colors (four status types, four data series) with no natural way to reduce the count.
- **Watch for:** it's rare to need all four at full volume simultaneously — let one complementary pair lead visually and keep the other pair in reserve for less frequent states.

## Practical Palette Archetypes

These aren't color-wheel relationships in the strict sense — they're patterns that recur across real products because of what they're *for*, not because of geometry on a wheel.

### 7. Neutral + Accent

Grayscale (or a warm/cool-tinted near-neutral) does all the structural work; exactly one color is left to mean "act here."

- `bg` `#F6F5F3` · `text` `#262421` · `accent` `#C6512F`
- **Use when:** building a content-first product — documentation, a dashboard, a reading app — where color should mean exactly one thing: an available action.
- **Watch for:** the single accent now has to do every job — primary button, active link, focus ring, selected state — and needs real discipline to keep meaning the same thing everywhere it appears.

### 8. Dark-Mode-First

Designed for a dark ground from the start, not a light theme run through a naive inverter (see [design-tokens-and-theming.md](design-tokens-and-theming.md) for why a real inversion needs its own token pass, not a filter).

- `bg` `#121212` · `text` `#EDEDEA` · `accent` `#C2F04C` · `ghost` `#4C86F0`
- **Use when:** the product is used in long, low-light sessions — terminals, editors, monitoring dashboards — where a light theme is genuinely the secondary case, not the primary one designed-down.
- **Watch for:** a saturated accent glows harder on a dark ground than the identical hex does on white — colors that felt balanced in a light palette often need desaturating when moved to a dark-first one.

### 9. Muted / Pastel

Every hue in the palette stepped back in saturation together, so nothing shouts because nothing is allowed to.

- `bg` `#FBF7F5` · `accent` `#C98A8A` · `tag` `#9AAE8D` · `tag 2` `#E3C978`
- **Use when:** the product is aiming for calm over urgency — consumer wellness, lifestyle, anything where "loud" would read as off-brand.
- **Watch for:** low contrast is the default failure mode here, not an edge case — check actual text contrast ratios harder than you'd check them for a bolder palette; pastel accents usually need dark text on them, not white, to stay legible.

### 10. High-Contrast

Near-black, near-white, and exactly one saturated color. Nothing to retreat to, nothing to hide behind.

- `bg` `#FFFFFF` · `text` `#0A0A0A` · `accent` `#E0261C`
- **Use when:** accessibility is a hard requirement, or the brand specifically wants to read as blunt and confident rather than soft.
- **Watch for:** there's no neutral middle ground to fall back on — every additional color you're tempted to add competes directly with the one accent instead of quietly supporting it.

### 11. Duotone

A single photographic-style two-color wash carried across the entire interface, the way a duotone print treatment works on an image.

- `bg` `#182A4A` · `text` `#F4E4DA` · `accent` `#F0765A`
- **Use when:** you want a distinct, unmistakably branded moment — a splash page, an about section, a single feature launch — not the resting state of a full application.
- **Watch for:** the whole effect depends on strictly two hues sharing the frame — introducing a third color, even a neutral, breaks the effect immediately and it stops reading as duotone.

### 12. Earth-Tone

Clay, olive, and sand — a palette borrowed from material and pigment (terracotta, moss, linen) rather than derived from color-wheel geometry.

- `bg` `#F6F0E6` · `text` `#3A2E22` · `accent` `#A4531F` · `tag` `#6E7B4C`
- **Use when:** the product wants to signal craft, sustainability, or something tactile and unpolished-on-purpose — food, outdoors, materials-forward brands.
- **Watch for:** low-saturation-by-design palettes can read as muddy under harsh screen brightness — check the palette on a phone outdoors in daylight, not just on a calibrated monitor.

## Related

- [design-tokens-and-theming.md](design-tokens-and-theming.md) — once a strategy is chosen, this is how its output becomes a real primitive/semantic token system, including the light/dark theming pattern.
- [avoiding-the-ai-generated-look.md](avoiding-the-ai-generated-look.md) — the default gradient tell described there is, in this catalog's terms, a complementary or analogous strategy applied without any of the discipline these entries describe.
