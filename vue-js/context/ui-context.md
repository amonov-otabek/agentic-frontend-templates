# UI Context — Design System

Machine-readable design system: tokens, theme, and the living component registry.
`context/ui-rules.md` says how to use these values; this file IS the values. Lock this
before the first UI task is built — retrofitting a design system is how features get
rebuilt (lesson learned from previous projects).

## Theme

- Mode: **light + dark** (default `system`). Toggled by `theme-provider.vue` (used in
  `App.vue`) which sets `.light`/`.dark` on `<html>` and persists to localStorage
  (`ui-theme`); surfaced via the `theme-toggle` widget.
- Personality: clean, neutral, professional — a data-dense dashboard built on
  shadcn-vue / reka-ui primitives (`style: reka-nova`, base color `neutral`).
- Component source: shadcn-vue, vendored under `shared/components/ui/*` (lucide icons).

## Tokens

Single source of truth: `src/styles/globals.css`. Raw values are CSS variables in
`:root` / `.dark` (OKLCH); Tailwind v4 `@theme inline` maps them to utility colors
(`bg-background`, `text-foreground`, `border-border`, …). Components reference utilities
by name and **never restate raw values**. Add/extend tokens here first.

```css
/* :root (light) — raw values in OKLCH */
--background:        oklch(1 0 0);          /* page background       */
--foreground:        oklch(0.145 0 0);      /* primary text          */
--card:              oklch(1 0 0);
--card-foreground:   oklch(0.145 0 0);
--popover:           oklch(1 0 0);
--popover-foreground:oklch(0.145 0 0);
--primary:           oklch(0.205 0 0);      /* primary actions       */
--primary-foreground:oklch(0.985 0 0);
--secondary:         oklch(0.97 0 0);
--secondary-foreground: oklch(0.205 0 0);
--muted:             oklch(0.97 0 0);
--muted-foreground:  oklch(0.556 0 0);      /* secondary text        */
--accent:            oklch(0.97 0 0);
--accent-foreground: oklch(0.205 0 0);
--destructive:       oklch(0.577 0.245 27.325); /* errors            */
--border:            oklch(0.922 0 0);
--input:             oklch(0.922 0 0);
--ring:              oklch(0.708 0 0);       /* focus ring            */
--chart-1..5:        neutral grayscale ramp (0.87 → 0.269)
--sidebar*:          dedicated sidebar surface tokens
--radius:            0.625rem;              /* base radius           */

/* Tailwind @theme inline derives: */
--font-sans:   'Geist Variable', sans-serif;   /* loaded via Google Fonts import */
--font-heading: var(--font-sans);
--radius-sm: calc(var(--radius) - 4px);   --radius-md: calc(var(--radius) - 2px);
--radius-lg: var(--radius);               --radius-xl: calc(var(--radius) + 4px);
```

`.dark` overrides every color token (e.g. `--background: oklch(0.145 0 0)`,
`--foreground: oklch(0.985 0 0)`, translucent borders). Never hardcode a dark variant —
use the token; it flips automatically under `.dark`.

### Color usage guide

| Use | Utility (token) |
| --- | --- |
| Page background | `bg-background` |
| Primary text | `text-foreground` |
| Secondary / muted text | `text-muted-foreground` |
| Primary actions / active state | `bg-primary` / `text-primary-foreground` |
| Cards / panels | `bg-card` / `text-card-foreground` |
| Borders / dividers / inputs | `border-border` / `border-input` |
| Focus ring | `ring-ring` (`outline-ring/50` base) |
| Error / destructive | `bg-destructive` / `text-destructive` |
| Sidebar surfaces | `bg-sidebar`, `text-sidebar-foreground`, … |
| Charts | `--chart-1` … `--chart-5` |

There is no dedicated success/warning token in the base palette — if a feature needs
one, add it here first (do not inline a green/amber hex).

## Typography

| Style | Token / class | Use |
| --- | --- | --- |
| Font family | `--font-sans` (Geist Variable) | all text; `--font-heading` aliases it |
| Heading | `font-semibold` + Tailwind size utility (e.g. `text-2xl`) | page/section titles |
| Body | base (`text-sm`/`text-base`) | default text |
| Caption | `text-xs text-muted-foreground` | metadata, hints |

Use Tailwind's type scale utilities; do not set raw `px` font sizes.

## Spacing & radius

- Spacing: Tailwind's default scale (`p-2`/`p-4`/`p-6`, `gap-4`, …) — multiples of
  `0.25rem`. Cards typically `p-6`, section gaps `gap-6`/`space-y-8`, form field gaps
  `gap-4`. No magic pixel margins.
- Radius: `rounded-sm|md|lg|xl` map to the `--radius-*` tokens above.

## Component registry (living document)

The shadcn-vue primitive library under `shared/components/ui/*` (accordion, alert,
button, card, dialog, dropdown-menu, form, input, table, sidebar, sonner, …) is the
**base layer** — reuse it; do not rebuild a primitive. This registry tracks **app-level
and feature components** built on top of those primitives. App widgets already present:
`shared/components/widgets/{icons, locale-switch, pager, sticky-header, theme-toggle}`.

Updated by Builders after every new component; checked before every new component;
audited by QA and `/review`. One entry per component:

```markdown
### ComponentName
- Path: `features/<feature>/components/component-name.vue` (or `shared/components/widgets/…`)
- Purpose: one line
- Props/emits: the public API surface
- Used by: pages/features using it
```

_(empty — first entry added by the first UI task)_

## Invariants

1. No visual value outside this file / `globals.css` tokens.
2. New tokens are added here (and in `globals.css`) first, then used — never
   inline-then-backfill.
3. Reuse the shadcn-vue primitive before building a new one; the registry never goes
   stale — building a component and not registering it is a QA rejection.
