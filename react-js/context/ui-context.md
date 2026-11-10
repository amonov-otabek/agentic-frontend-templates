# UI Context — Design System

Machine-readable design system: tokens, theme, and the living component registry.
`context/ui-rules.md` says how to use these values; this file IS the values. The tokens
below are the real ones defined in `src/styles/globals.css` (Tailwind v4 `@theme` +
`:root` / `.dark` palettes). Lock this before retrofitting any visual value — that is
how features get rebuilt.

## Theme

- Mode: **light + dark** (default `system` via `next-themes`, `class` strategy).
- Personality: clean, neutral, professional dashboard. Achromatic base palette
  (grayscale via OKLCH) with a single dark "primary" for actions; data viz uses a
  monochrome chart ramp.
- Primitives: **shadcn** (`style: base-nova`) over `@base-ui/react`; icons via
  `lucide-react`. Font: **Geist Variable** (`@fontsource-variable/geist`).
- Source of truth: `src/styles/globals.css`. Tokens are CSS variables surfaced to
  Tailwind through `@theme inline`; components reference them as Tailwind classes
  (`bg-background`, `text-muted-foreground`, `rounded-md`) — never raw values.

## Tokens

Single source of truth. Defined once in `src/styles/globals.css` and referenced
everywhere by name — components never restate values. Colors are **OKLCH**; the base
palette is achromatic (chroma 0). Light values shown; `.dark` overrides each.

```css
/* :root (light) — see .dark for the dark counterparts */
--background: oklch(1 0 0);            /* page background */
--foreground: oklch(0.145 0 0);        /* primary text */
--card: oklch(1 0 0);
--card-foreground: oklch(0.145 0 0);
--popover: oklch(1 0 0);
--popover-foreground: oklch(0.145 0 0);
--primary: oklch(0.205 0 0);           /* primary actions */
--primary-foreground: oklch(0.985 0 0);
--secondary: oklch(0.97 0 0);
--secondary-foreground: oklch(0.205 0 0);
--muted: oklch(0.97 0 0);
--muted-foreground: oklch(0.556 0 0);  /* secondary text */
--accent: oklch(0.97 0 0);
--accent-foreground: oklch(0.205 0 0);
--destructive: oklch(0.577 0.245 27.325); /* errors / destructive */
--border: oklch(0.922 0 0);
--input: oklch(0.922 0 0);
--ring: oklch(0.708 0 0);              /* focus ring */
--chart-1..5: oklch(0.87…0.269 0 0);   /* monochrome data ramp */
--sidebar, --sidebar-foreground, --sidebar-primary, --sidebar-accent, --sidebar-border, --sidebar-ring;

/* Radii — derived from one base */
--radius: 0.625rem;                    /* base (10px) */
--radius-sm: calc(var(--radius) * 0.6);
--radius-md: calc(var(--radius) * 0.8);
--radius-lg: var(--radius);
--radius-xl / 2xl / 3xl / 4xl: calc(var(--radius) * 1.4 … 2.6);

/* Typography */
--font-sans: 'Geist Variable', sans-serif;
--font-heading: var(--font-sans);
```

Note: there is **no green/amber success/warning token** in the base palette —
`--destructive` is the only semantic color. Status badges use `secondary`/`muted`/
`destructive` variants. If a project needs success/warning, add the tokens here first.

### Color usage guide

| Use | Token (Tailwind class) |
| --- | --- |
| Page background | `bg-background` |
| Primary text | `text-foreground` |
| Secondary / muted text | `text-muted-foreground` |
| Surfaces / panels | `bg-card` / `bg-popover` |
| Primary actions, active states | `bg-primary` / `text-primary-foreground` |
| Secondary / subtle actions | `bg-secondary` / `bg-accent` |
| Borders / dividers / inputs | `border-border` / `border-input` |
| Focus ring | `ring-ring` |
| Errors / destructive | `text-destructive` / `bg-destructive` |
| Data viz series | `--chart-1` … `--chart-5` |
| Sidebar chrome | `bg-sidebar`, `text-sidebar-foreground`, … |

## Typography scale

Font: **Geist Variable** (`font-sans`, also used for headings). Sizes via Tailwind's
text scale; weights via `font-medium` / `font-semibold` / `font-bold`.

| Style | Tailwind | Use |
| --- | --- | --- |
| Heading 1 | `text-2xl font-bold` | page titles |
| Heading 2 | `text-lg font-semibold` | section titles |
| Body | `text-sm` (default) | default text |
| Caption | `text-xs text-muted-foreground` | metadata, hints, labels |

## Spacing & radii

- Spacing: Tailwind default scale (`gap-2`/`gap-3`/`gap-6`, `space-y-6` for page
  sections, `p-4`/`p-6` for cards). Observed defaults: section stacks `space-y-6`,
  intra-section `space-y-2`/`space-y-3`, control gaps `gap-2`.
- Radii: use the `--radius-*` tokens (`rounded-md` / `rounded-lg`); never literal px.
- Icon sizing via Tailwind (`size-4` for inline controls).

## Component registry (living document)

Updated by Builders after every new component; checked by Builders before every new
component; audited by QA and `/review`. One entry per component.

shadcn primitives already live under `src/shared/components/ui/*` (button, card, table,
dialog, input, badge, tabs, sidebar, sonner, …) — reuse them; do not re-create. Register
**feature** components (`src/features/<name>/components/*`) and shared **widgets**
(`src/shared/components/widgets/*`) here.

```markdown
### ComponentName
- Path: `src/features/<feature>/components/ComponentName.tsx`
- Purpose: one line
- Variants/props: the public API surface
- Used by: pages/features using it
```

_(empty — first entry added by the first UI task; the shadcn `ui/*` primitives and the
existing widgets are the reusable baseline.)_

## Invariants

1. No visual value outside this file / `globals.css`.
2. New tokens are added here (and in `globals.css`) first, then used — never
   inline-then-backfill.
3. The registry never goes stale: building a component and not registering it is a QA
   rejection.
4. Reuse `shared/components/ui` primitives before authoring a new component; compose with
   `cn()` (`shared/libs/utils`).
