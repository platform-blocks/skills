# Platform Blocks skills

[Agent Skills](https://skills.sh) for building with [Platform Blocks](https://platform-blocks.com) — a cross-platform React Native UI library. Each skill teaches an AI coding agent (Claude Code, Cursor, and other skills-compatible agents) the real APIs, working patterns, and pitfalls of one area of the library, verified against the source.

## Skills

| Skill | Covers |
| --- | --- |
| `platform-blocks-setup` | Installing `@platform-blocks/ui`, the true dependency set, provider wiring, Metro/Jest/TypeScript configuration in consumer apps, flash-free dark mode on web |
| `platform-blocks-theming` | Theme anatomy, custom palettes, light/dark/auto switching, surfaces & elevation, variant color resolution, web CSS variables |
| `platform-blocks-layout` | Flex/Row/Column/Grid, spacing props, responsive breakpoints, AppShell blueprints, Cards and Surfaces |
| `platform-blocks-forms` | Form/ControlField composition, every input component's value/onChange contract, validation patterns, keyboard handling |
| `platform-blocks-charts` | All 25 chart types in `@platform-blocks/charts`, data shapes, theming via `hostThemeBridge`, interactions, streaming data |

## Installation

Install a skill with the [skills.sh](https://skills.sh) CLI:

```bash
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-setup
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-theming
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-layout
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-forms
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-charts
```

Each skill is a directory under [`skills/`](./skills) with a `SKILL.md` (workflow + pitfalls) and a `references/` folder (`api.md` for the full API surface, `patterns.md` for complete copy-paste examples).

## LLM-friendly docs

Platform Blocks also publishes its full documentation for LLMs at [platform-blocks.com/llms.txt](https://platform-blocks.com/llms.txt) — an index of per-page Markdown covering every component, chart, hook, and guide.

## License

MIT
