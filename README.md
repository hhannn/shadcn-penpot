# shadcn-penpot

A [shadcn/ui](https://ui.shadcn.com) (base-vega) component library rebuilt in [Penpot](https://penpot.app), plus agent skills for moving between the design file and code.

## What's in the Penpot file

- **66 components** (~200 library components) in docs order, each in its own titled section, with variant properties matching the React props (e.g. Button: `variant` × `size` × `icon`).
- **Design tokens**
  - `shadcn/base`: radius scale, Tailwind text scale, line height, letter spacing, font weights, font families, breakpoints, container widths (3xs–7xl), shadows, typography presets (text-xs…9xl, h1–h4, p, lead, large, small, muted, code)
  - `shadcn/light` / `shadcn/dark`: 36 color tokens each (shadcn CSS variables + chart + sidebar)
  - Themes: `mode / light` and `mode / dark`
- Layer names match React part names (`CardHeader`, `DialogContent`, …).

## Skills

| Skill | Purpose |
| --- | --- |
| [`penpot-shadcn-design-to-code`](skills/penpot-shadcn-design-to-code/SKILL.md) | Turn a Penpot board built with this library into React (shadcn/ui) or SvelteKit (shadcn-svelte) code |
| [`penpot-shadcn-code-to-design`](skills/penpot-shadcn-code-to-design/SKILL.md) | Build or update a library component in Penpot from React (cva) or Svelte (`tv()`) source |

Both require the Penpot MCP server.
