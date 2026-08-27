# Avoiding the "Obviously AI-Generated" Look

AI design and coding tools make it trivial to produce a plausible-looking interface in seconds. The problem is exactly that ease: without a deliberate point of view pushing back, output from different tools and different prompts converges on the same narrow set of "safe, modern-looking" defaults — which is precisely what makes it recognizable as generated rather than designed. This entry names the actual tells and what breaks the pattern, for anyone using AI tools to build UI (which, at this point, is most UI work).

## Why it converges

A model producing a design is optimizing for "broadly plausible and inoffensive," not "correct for this specific product." Left without a strong content- or brand-driven constraint, that optimization regresses toward the population mean of whatever it's seen most: current SaaS landing pages, current component-library defaults, current dashboard templates. The tells below aren't bad in isolation — they're bad because they show up *identically* regardless of what the product actually is, which is the giveaway that the layout was filled in rather than derived from the content.

## The recognizable tells

- **The purple/violet-to-blue gradient**, on backgrounds, buttons, and hero sections, as the default "modern" signal. It's become the visual equivalent of stock-photo handshakes.
- **Glassmorphism used decoratively** — frosted-blur cards floating over a gradient, applied because it reads as current, not because translucency is communicating layering or depth for a real reason.
- **An icon-in-a-circle above every card**, and an emoji next to every heading, regardless of whether either is adding information.
- **The same three-column feature-card grid** repeated at every scroll section, independent of whether the content underneath actually has three parallel items.
- **The centered hero template**: giant heading, one-line subheading, two pill-shaped buttons, all center-aligned — applied whether or not centering serves the content's actual reading order.
- **Uniform `rounded-2xl` + soft shadow on every surface** — cards, buttons, inputs, images all get the identical treatment, which flattens hierarchy instead of establishing it. If everything is emphasized the same way, nothing is.
- **A single default type scale** (usually Inter or a system-ui stack) with hierarchy carried only by size, not weight, spacing, or color — technically legible, visually flat.

None of these are wrong as isolated choices. A frosted card can be the right call; a gradient hero can be the right call. What makes them a tell is applying them by default, everywhere, without a reason specific to the thing being designed.

## What actually breaks the pattern

**Start from content, not from a template's empty slots.** Before laying anything out, name the one thing this screen needs to communicate and let the layout follow from that. A screen that's forced to fill "hero, three feature cards, testimonial, footer CTA" regardless of what it's for is what produces filler copy and decorative icons in the first place — there's nothing real to put in those slots, so generic content and generic decoration fill the gap.

**Commit to a real, constrained palette.** Two or three colors used with intent — one dominant, one accent, deployed consistently for the same *meaning* each time (e.g., the accent always means "primary action," never decoration) — reads as designed. A gradient used as decorative background reads as generated, because it isn't standing for anything.

**Vary weight and shape on purpose.** If every card has the same corner radius and the same shadow, hierarchy has to be carried entirely by position, and most layouts don't have enough positional variety to do that job. Give the one thing that matters most on a screen a visually distinct treatment from everything else on it — different weight, different scale, different surface — instead of matching the template's uniform treatment.

**Make icons and emoji earn their place.** An icon that indicates status, category, or a real affordance (a checkmark meaning "done," a lock meaning "restricted") is doing work. An icon sitting decoratively above a heading that already says the same thing in words is not. When in doubt, remove it and check whether anything was actually lost.

**Treat typography as a hierarchy decision, not a font choice.** The font matters less than whether size, weight, color, and spacing are used *together*, deliberately, to make the reading order obvious at a glance — not just technically correct heading tags at their default browser sizes.

## Accessibility is where the shortcut shows

This is a design-system concern independent of the "AI look," but it's exactly where copying a generated template uncritically causes real harm, not just a stylistic tell: templates are usually built to look right in their default state and are frequently missing the states that don't show up in a static screenshot — focus rings, sufficient contrast in both light and dark themes, keyboard-reachable interactive elements, `alt` text on meaningful images. A hover state existing while the corresponding focus state doesn't is a specific, common symptom of visual polish being prioritized over usability during a fast generation pass. Checking for these explicitly, rather than assuming the template handled them, is part of the same discipline as checking for the visual tells above.

## A quick self-review before shipping

- Would this layout look different if the actual content were swapped out for something with a different shape (more items, longer copy, no image)? If not, the layout was filled, not designed.
- Is there a gradient, icon, or decorative element on the page that isn't standing for anything specific? Remove it and see if the page is worse.
- Does one thing on this screen look more important than everything else, on purpose? If everything has equal visual weight, nothing has been prioritized.
- Do focus states, contrast ratios, and keyboard navigation work, or only the states visible in the default screenshot?

## Related

- [prompts/ui-ux-designer.md](../prompts/ui-ux-designer.md) — a ready-to-use system prompt for a UI/UX design assistant; the critique habits above apply directly to reviewing whatever it produces.
