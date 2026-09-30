<p align="center">
  <a href="https://plocks.dev/" rel="noopener" target="_blank"><img width="75" height="75" src="https://raw.githubusercontent.com/platform-blocks/plocks/HEAD/apps/docs/assets/favicon.png" alt="plocks logo"/></a>
</p>

<h1 align="center">plocks Skills</h1>

<p align="center">
  Agent Skills that teach AI coding agents such as Claude Code and Cursor how to build with <a href="https://plocks.dev">plocks</a>.
</p>

## Install

```bash
npx skills add https://github.com/platform-blocks/skills --skill plocks-setup
```

Swap in any skill name from the table below. Installs use the [skills.sh](https://skills.sh) CLI.

## Skills

| Skill | Covers |
| --- | --- |
| `plocks-setup` | Installation, peer dependencies, provider, Metro and Jest config |
| `plocks-theming` | Custom palettes, light/dark mode, surfaces, web CSS variables |
| `plocks-layout` | Flex, Grid, spacing, breakpoints, AppShell |
| `plocks-forms` | Form, every input component, validation, keyboard handling |
| `plocks-charts` | All 25 chart types in `@plocks/charts` |
| `plocks-feedback-overlays` | Toasts, dialogs, popovers, menus, tooltips, loaders |
| `plocks-data-display` | DataTable, Table, Tree, Timeline, Accordion, badges |
| `plocks-navigation` | Tabs, Stepper, Pagination, Spotlight, Expo Router |

Each skill has a `SKILL.md` and a `references/` folder with the API, prop tables, and complete example screens. For anything outside these areas, point your agent at [plocks.dev/llms.txt](https://plocks.dev/llms.txt).

## Contributing

`references/props.md` and `references/icons.md` are generated from the library source by `npm run skills:generate` in the [main repository](https://github.com/platform-blocks/plocks). Edit the other files here directly.
