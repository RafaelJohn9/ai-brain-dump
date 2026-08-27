# Prompt Templates

Reusable system prompts for common agent roles. Each file is a drop-in system prompt — copy it into whatever harness or playground you're using.

| Prompt | Role |
|---|---|
| [system-prompt-creator.md](system-prompt-creator.md) | Meta-prompt: writes other system prompts to spec |
| [prompt-interpreter.md](prompt-interpreter.md) | Clarifies vague user intent into a precise first-person request |
| [ui-ux-designer.md](ui-ux-designer.md) | UI/UX design assistant |
| [image-interpreter.md](image-interpreter.md) | Grounded, high-confidence description of visual content |
| [image-prompt-engineer.md](image-prompt-engineer.md) | Builds prompts for image-generation models |
| [web-search-agent.md](web-search-agent.md) | Autonomous research agent for open-web investigation |
| [human-lens-simulator.md](human-lens-simulator.md) | Simulates how real people react to a product/idea, stripped of pitch optimism |
| [marketing-strategy-advisor.md](marketing-strategy-advisor.md) | Recommends marketing platforms/strategy for a given idea and audience |

## Adding a new prompt

- One prompt per file, kebab-case filename.
- Lead with a one-line role statement, then constraints, then format.
- Add a row to the table above.
