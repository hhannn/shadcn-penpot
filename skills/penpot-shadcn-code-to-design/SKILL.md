---
name: penpot-shadcn-code-to-design
description: Build or update a shadcn/ui component in the Penpot shadcn library from React (cva) or SvelteKit (shadcn-svelte, tailwind-variants) source, with variants, tokens and a docs section.
---

# shadcn component code (React or Svelte) → Penpot component

Use when the user gives shadcn component source — React (shadcn/ui) or Svelte (shadcn-svelte) — or names a component to build/update in the Penpot file "shadcn" via the Penpot MCP.

## 1. Source of truth
1. User-pasted code wins.
2. React: `ui.shadcn.com/r/styles/base-vega/<name>.json` (search first; fetch only URLs that appeared in results), fallback `new-york-v4`, then the docs page.
3. Svelte: shadcn-svelte component source (`$lib/components/ui/<name>/*.svelte`, from the shadcn-svelte docs/GitHub) or the user's project files.
4. Tell the user which parts are exact source vs derived. Use the docs demo content for the demo.

## 2. Read the variants (both frameworks express the same thing)
- React: `cva(base, { variants, defaultVariants })` → base classes + variant classes. Part functions (`function CardHeader`) = parts.
- Svelte: `tv({ base, variants, defaultVariants })` from `tailwind-variants` in `<script module>`; parts are files (`card-header.svelte`) exported as `Card.Header` → **Penpot part name `CardHeader`** (family prefix + part). Root (`Card.Root`) → `Card`.
- Class props: React `className` / Svelte `class` — same Tailwind strings.
- `data-slot` values confirm part names in both.

## 3. Mapping rules (Tailwind → Penpot)
- Spacing: value × 4px. Board paddings include the 1px border (`px-2.5` + `border` → 11).
- `h-*`/`size-*` → fixed sizes; `w-fit` → hug (`flex.horizontalSizing='auto'` AFTER any resize).
- `rounded-*` → bind `radius.*` on all four corners; `rounded-[min(var(--radius-md),Npx)]` → `radius.md`; fixed values (`rounded-[4px]`, `rounded-xs`) stay raw (say so).
- Colors → bind `color.*` for fill AND stroke (`applyToken(tok,['fill'])`, `['strokeColor']`). Opacity modifiers need alpha tokens (`color.foreground-10`, `muted-50`, `primary-20`, `destructive-10`, `foreground-70`); a plain color token resets opacity to 1.
- `ring-1 ring-foreground/10` → outer stroke with `color.foreground-10`. Focus/invalid rings → drop-shadow spread 3 (not tokenizable).
- Shadows: set on shapes with exact Tailwind values (tokens can't hold negative spread); bind `shadow.xs`/`2xs` (exact) when used.
- Text: bind `typography.text-*` (size + line height + sans, weight unset) and `font-weight.*`; shadcn typography classes → `typography.h1…code`. `tracking-*` is computed px (`{text.X} * {tracking.Y}`), `leading-*` → `leading.*`.
- `[&_svg]:size-N` → icon instance size N.
- Variants → Penpot variant properties with the same names/values. Interaction-only states may become a `state` property. `has-data-[icon=…]` → `icon` property (none/inline-start/inline-end) with adjusted padding.
- Layer names = React part names (also for Svelte sources, via the mapping above).

## 4. Build procedure (Penpot MCP `execute_code`)
1. Wake check with a trivial call. If `storage` helpers are gone (plugin reloaded), re-create: token lookup (`penpotUtils.findTokenByName`), `box(name,{dir,gap,pt..,align,justify,fill,hs,vs})`, `text(str,{size,weight,color,lh})` (Geist), `tokFill`, `iconInst(name,size,color)` (resize the icon board AND its inner `svg` group), section helpers, `ORDER` + `restack()`, `strays()`, `audit()`.
2. **Open the components page first** (`penpot.openPage(...)`): edits on a non-active page are rejected or silently ignored.
3. One main per variant combo → `library.local.createComponent([board])` → `penpotUtils.createVariantContainer(entries)`. Extending a set: `vc.variants.addProperty()` + `renameProperty(i,name)`, `component.setVariantProperty(i,value)`, `component.addVariant()` (rename the placeholder value immediately).
4. Reuse library components inside composites (Button, Input, Label, Avatar, icons) as instances. Find variants through the variant container (`variantComponents()` + `variantProps`), not by name.
5. Wrap in a section (`makeSection(Name, docs description, [...])`, rows for wide sets), `restack()`, verify with `export_shape`.

## 5. Known Penpot quirks
- Set instance text AFTER appending the instance; re-check overrides.
- Can't append children into a component copy — wrap it (e.g. `SidebarMenuItem` board + absolute badge).
- Negative `gap` is invalid — use negative child margins.
- Multi-corner radius tokens on instances can silently fail — apply per corner with short delays and verify `tokens`.
- Swapping an icon resets its color — re-apply stroke color + token.
- Instances keep size overrides when a main is resized — sweep copies and reset height (and width for icon-only sizes).
- Failed scripts leave partial shapes — run `strays()` and clean up.
- Long scripts can time out — batch, keep operations idempotent; if the tab is suspended, ask the user to focus it.
- Changing a shared main (e.g. Button) affects every section — re-export a couple of dependent sections.

## 6. Report
What was built (properties, variant counts), source per part (exact/derived, React/Svelte), raw untokenized values, and open decisions.
