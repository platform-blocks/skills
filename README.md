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
| `platform-blocks-feedback-overlays` | Toast/ToastProvider, Dialog & bottom sheets, Popover/Menu/Tooltip/ContextMenu, Alert, Loader/Skeleton/Progress/Ring/LoadingOverlay, useDisclosure |
| `platform-blocks-data-display` | DataTable (sort/filter/paginate/select, server-side), the Table compound, DataList, Tree, Timeline, Accordion, Badge/Chip/Avatar/Indicator |
| `platform-blocks-navigation` | Tabs, Breadcrumbs, Pagination, Stepper, Spotlight command palette, TableOfContents, Link, hotkeys, and Expo Router integration |

## Installation

Install a skill with the [skills.sh](https://skills.sh) CLI:

```bash
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-setup
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-theming
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-layout
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-forms
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-charts
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-feedback-overlays
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-data-display
npx skills add https://github.com/platform-blocks/skills --skill platform-blocks-navigation
```

Each skill is a directory under [`skills/`](./skills) with a `SKILL.md` and a `references/` folder:

| File | Written by | Contents |
| --- | --- | --- |
| `SKILL.md` | hand | Workflow, decision tables, pitfalls verified against source |
| `references/api.md` | hand | The curated API — how pieces compose, union types, defaults that bite |
| `references/props.md` | **generated** | Exhaustive prop tables, extracted from the component sources |
| `references/icons.md` | **generated** | Every `name` the built-in `Icon` registry accepts |
| `references/patterns.md` | hand | Complete copy-paste screens |

Verified against `@platform-blocks/ui` 1.0.0 and 1.0.1. Where the two versions behave differently — notably the optional-dependency set — both are documented.

## Keeping these in sync

The generated files and the drift checks live in the
[monorepo](https://github.com/platform-blocks/platform-blocks) as
`scripts/generate-skills.ts`. With this repo cloned next to it:

```bash
npm run demos:generate     # refresh the extracted prop data
npm run skills:generate    # write props.md + icons.md into ../skills
npm run skills:check       # fail if generated files are stale, or a skill
                           # names a symbol or icon the library no longer has
```

`skills:check` runs as part of `verify:packages`, so a release cannot ship a
library whose skills teach an API that no longer exists. It re-parses every
`import { … } from '@platform-blocks/…'` in every fenced code block and every
`<Icon name="…">`, and fails on anything unresolvable. If this repo is not
checked out beside the monorepo, the check skips rather than blocking the build.

Hand-written files are never overwritten — they are checked instead. Prop tables
go stale silently and are dull to maintain, so they are generated; everything
that required judgement stays hand-written.

## Beyond the skills

These eight skills go deep on eight areas; they still do not cover all 97 components. Every `SKILL.md` therefore ends with a section pointing at the generated docs, so an agent that hits something outside a skill's scope fetches the real API instead of guessing:

- [platform-blocks.com/llms.txt](https://platform-blocks.com/llms.txt) — index of every page, one line each
- `https://platform-blocks.com/llms/components/<Name>.md` — one component or chart, with its full prop table and examples
- `https://platform-blocks.com/llms/hooks/<useName>.md` — one hook
- [platform-blocks.com/llms-full.txt](https://platform-blocks.com/llms-full.txt) — everything in one file

## License

MIT
