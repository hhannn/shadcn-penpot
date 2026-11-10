---
name: penpot-shadcn-design-to-code
description: Turn a Penpot board built with the shadcn/ui Penpot library into React (shadcn/ui) or SvelteKit (shadcn-svelte) code using the project's components and Tailwind tokens.
---

# Penpot (shadcn library) → React or SvelteKit

Use when the user points at a Penpot board, frame or selection made from the shadcn/ui component library (file "shadcn", page "components") and wants code.

## 0. Pick the target framework
- If the user says React/Next.js → **React** (shadcn/ui). If they say Svelte/SvelteKit → **Svelte** (shadcn-svelte, Svelte 5 runes, Bits UI).
- If a repo is available, detect: `svelte.config.js` / `$lib/components/ui` → Svelte; `next.config.*` / `components.json` with `"tsx": true` → React. Read `components.json` aliases for import paths.
- If still unknown, ask once. Never emit both unless asked.

## 1. Library conventions this relies on
- Every component is a Penpot library component named after the React part (Button, CardHeader, DropdownMenuItem…). Variant property names/values equal the props (`variant`, `size`, `orientation`, `side`, `isActive`, `checked`…).
- Button has an `icon` property: `none | inline-start | inline-end` → place the icon before/after the label with `data-icon="inline-start|inline-end"` on the icon.
- Display-only `state` variants (default/focus/hover/open/disabled/invalid/filled/placeholder): `disabled` → `disabled`, `invalid` → `aria-invalid`, `open` → parent primitive open state, `focus/hover` → ignore.
- Layer names inside composites are part names, so the layer tree is the component tree.
- Tokens (sets `shadcn/base`, `shadcn/light`, `shadcn/dark`; theme group `mode`):
  - `color.<name>` → same-named Tailwind color (`text-*` on text, `bg-*` on boards, `border-*`/`ring-*` on strokes). Alpha helpers: `color.foreground-10` → `ring-foreground/10`, `color.muted-50` → `bg-muted/50`, `color.primary-20` → `bg-primary/20`, `color.destructive-10` → `bg-destructive/10`, `color.foreground-70` → `…/70`.
  - `radius.*` → `rounded-*`. `shadow.*` → `shadow-*`, `inset-shadow.*` → `inset-shadow-*` (descriptions hold exact values). `container.*` → `w-*`/`max-w-*`. `breakpoint.*` → frame width = which breakpoint the screen targets (use responsive prefixes accordingly).
  - Typography: `typography.text-*` → `text-*`; shadcn styles `typography.h1…h4|p|lead|large|small|muted|code` → the shadcn Typography class strings in each token's description. `font-weight.*` → `font-*`, `font-family.sans|mono` → `font-sans|font-mono`, `leading.*` → `leading-*`, `tracking.*` → `tracking-*`.

## 2. Read the design (Penpot MCP)
1. Use `execute_code`. **Edits and some reads only work on the active page** — if the target is on another page, `penpot.openPage(page)` first. Locate the board (`penpotUtils.findShapeById`, or by name on `penpot.root`), then `export_shape` to see it.
2. Walk with `penpotUtils.analyzeDescendants` into an outline:
   - Component copies (`isComponentCopyInstance()` + `component()`): component name, `component().variantProps` (for variant sets resolve via the variant container), text overrides (`type==='text'` → `characters`), swapped icons (child names ending in `Icon` → Lucide icon of the same name).
   - Flex boards: `flex.dir` → `flex-row|flex-col`, gaps → `gap-*`, paddings → `p*-*`, align/justify → `items-*`/`justify-*`, `horizontalSizing:'fill'` → `w-full`/`flex-1`, `'auto'` → `w-fit`, absolute children → `absolute` + offsets.
   - Bound tokens (`shape.tokens`) win over raw values; only use arbitrary values (`[…px]`) for unbound properties and list them.
3. px → spacing: divide by 4 (8 → `2`, 10 → `2.5`). Component paddings include the 1px border (11px = `px-2.5` + border).

## 3. Emit code

### React (shadcn/ui, Base UI style)
- `import { Button } from "@/components/ui/button"`; group a family: `import { Card, CardHeader, CardTitle } from "@/components/ui/card"`. Icons: `import { ArrowRightIcon } from "lucide-react"`.
- `className`, `render={<Button …/>}` for trigger composition (Base UI; Radix projects use `asChild`).
- Output one `.tsx` file, default export named after the board in PascalCase.

### SvelteKit (shadcn-svelte, Svelte 5)
- Single-part components: `import { Button } from "$lib/components/ui/button/index.js";` (Button, Input, Label, Badge, Checkbox, Switch, Textarea, Separator, Skeleton, Kbd…).
- Multi-part families use namespaces: `import * as Card from "$lib/components/ui/card/index.js";` and map part names by stripping the family prefix: `Card` → `<Card.Root>`, `CardHeader` → `<Card.Header>`, `DropdownMenuItem` → `<DropdownMenu.Item>`, `SidebarMenuButton` → `<Sidebar.MenuButton>`.
- Icons: `import ArrowRightIcon from "@lucide/svelte/icons/arrow-right";` → `<ArrowRightIcon />`.
- `class=` instead of `className`. Same `variant`/`size` prop names.
- Trigger composition uses the child snippet: `<DropdownMenu.Trigger>{#snippet child({ props })}<Button {...props} variant="outline">Open</Button>{/snippet}</DropdownMenu.Trigger>`.
- State with runes (`let open = $state(false)`, `bind:value`, `bind:checked`) only where the design implies interactivity.
- Output one `.svelte` component (`<script lang="ts">` + markup), or a `+page.svelte` if the board is a full page.
- If a component doesn't exist in shadcn-svelte (newer shadcn-only parts), say so and build it from Tailwind classes with the same tokens.

## 4. Check
Every instance in the board is represented, texts match, icons imported, no hard-coded hex unless the design was untokenized. Finish with a short list of assumptions (dropped display states, arbitrary values, missing components).
