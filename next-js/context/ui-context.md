# UI Context — Design System

Machine-readable design system: tokens, theme, and the living component registry.
`context/ui-rules.md` says how to use these values; this file IS the values. Lock this
before UI work — retrofitting a design system is how features get rebuilt. The single
source for all tokens is `src/styles/globals.css` (Tailwind v4 `@theme inline`); this
file documents them and what each is for. Components never restate raw values.

## Theme

- Mode: **light + dark** (default light; toggled by `next-themes` via the `.dark` class).
- Personality: clean, neutral, professional dashboard — grayscale base, content-forward,
  low chrome. Color is reserved for state (destructive) and data (charts).
- Color space: **oklch** CSS variables, defined per mode in `:root` / `.dark`.
- Primitives: shadcn-style components on `@base-ui/react`; icons via `lucide-react`;
  charts via `recharts`.

## Tokens

Single source of truth — defined in `src/styles/globals.css` and referenced by name
through Tailwind utilities (`bg-background`, `text-muted-foreground`, `rounded-lg`, …).
Values shown are the light-mode `:root` set; `.dark` overrides the same variable names.

```css
/* Surfaces & text (oklch) */
--background: oklch(1 0 0);              /* page background */
--foreground: oklch(0.145 0 0);         /* primary text */
--card: oklch(1 0 0);                    /* card surface */
--card-foreground: oklch(0.145 0 0);
--popover: oklch(1 0 0);
--popover-foreground: oklch(0.145 0 0);

/* Brand / actions */
--primary: oklch(0.205 0 0);            /* primary actions */
--primary-foreground: oklch(0.985 0 0);
--secondary: oklch(0.97 0 0);
--secondary-foreground: oklch(0.205 0 0);

/* Muted & accent */
--muted: oklch(0.97 0 0);
--muted-foreground: oklch(0.556 0 0);   /* secondary text */
--accent: oklch(0.97 0 0);
--accent-foreground: oklch(0.205 0 0);

/* State */
--destructive: oklch(0.577 0.245 27.325); /* errors / destructive */

/* Lines & focus */
--border: oklch(0.922 0 0);
--input: oklch(0.922 0 0);
--ring: oklch(0.708 0 0);               /* focus ring */

/* Charts */
--chart-1: oklch(0.87 0 0);  --chart-2: oklch(0.556 0 0);  --chart-3: oklch(0.439 0 0);
--chart-4: oklch(0.371 0 0); --chart-5: oklch(0.269 0 0);

/* Sidebar surface set */
--sidebar: oklch(0.985 0 0); --sidebar-foreground: oklch(0.145 0 0);
--sidebar-primary: oklch(0.205 0 0); --sidebar-accent: oklch(0.97 0 0);
--sidebar-border: oklch(0.922 0 0); --sidebar-ring: oklch(0.708 0 0);

/* Radius base (radii derived from it in @theme) */
--radius: 0.625rem; /* 10px */
```

Derived radii (from `@theme inline`): `--radius-sm` (×0.6 ≈ 6px), `--radius-md` (×0.8 ≈ 8px),
`--radius-lg` (= 10px), `--radius-xl` (×1.4), `--radius-2xl` (×1.8), `--radius-3xl`/`4xl`.

Fonts (from `@theme`): `--font-sans` (UI default), `--font-mono`, `--font-heading` (= sans).

### Color usage guide

| Use | Token |
| --- | --- |
| Page background | `--background` |
| Card / popover surface | `--card` / `--popover` |
| Primary text | `--foreground` |
| Secondary / muted text | `--muted-foreground` |
| Primary actions (buttons, active nav) | `--primary` (text `--primary-foreground`) |
| Secondary actions / chips | `--secondary` (text `--secondary-foreground`) |
| Subtle backgrounds / hover | `--muted` / `--accent` |
| Borders, dividers, inputs | `--border` / `--input` |
| Focus ring | `--ring` |
| Errors / destructive actions | `--destructive` |
| Data viz | `--chart-1` … `--chart-5` |
| App sidebar | `--sidebar*` set |

There is no separate success/warning token — convey success via copy/toasts (`sonner`)
and reserve `--destructive` for errors. Add new state tokens here first if needed.

## Typography scale

Use `next/font`-loaded `--font-sans` for everything; weights via Tailwind utilities.

| Style | Size / weight | Use |
| --- | --- | --- |
| Heading 1 | `text-2xl` / `font-semibold` | page / auth titles |
| Heading 2 | `text-lg` / `font-semibold` | section + card titles |
| Body | `text-sm` / `font-normal` | default UI text |
| Caption | `text-xs` / `font-normal` | metadata, hints, helper text |

Secondary text uses `text-muted-foreground`. Mono (`--font-mono`) only for codes/tokens.

## Spacing

Tailwind spacing scale (4px base): `2 / 4 / 6 / 8 / 12 / 16 / 24` etc. Conventions:
card padding `p-6` (24px), section gaps `gap-8`/`space-y-8` (32px), form field gaps
`space-y-4` (16px), inline element gaps `gap-2` (8px). No magic pixel values.

## Component registry (living document)

Updated by Builders after every new component; checked by Builders before every new
component; audited by QA and `/review`. One entry per component:

```markdown
### ComponentName
- Path: `features/<name>/components/...` or `shared/components/...`
- Purpose: one line
- Variants/props: the public API surface
- Used by: pages/features using it
```

### Primitives (`shared/components/ui/`)

shadcn-style primitives are already vendored — **reuse, do not rebuild**. Available:
accordion, alert, alert-dialog, aspect-ratio, avatar, badge, breadcrumb, button,
button-group, calendar, card, carousel, chart, checkbox, collapsible, combobox, command,
context-menu, dialog, direction, drawer, dropdown-menu, empty, field, hover-card,
input, input-group, input-otp, item, kbd, label, menubar, native-select, navigation-menu,
pagination, popover, progress, radio-group, resizable, scroll-area, select, separator,
sheet, sidebar, skeleton, slider, sonner, spinner, switch, table, tabs, textarea,
toggle, toggle-group, tooltip.

### Shared widgets (`shared/components/widgets/`)

icons, locale-switch, pager, sticky-header, theme-toggle.

_(Feature components — e.g. `identity/components/sign-in-form`, `message/components/message-list`
— are registered here by the Builder as each is built/changed.)_

## Invariants

1. No visual value outside the tokens in `styles/globals.css` (documented here).
2. New tokens are added to `globals.css` (and noted here) first, then used — never
   inline-then-backfill.
3. Reuse a `shared/components/ui` primitive before building a new component; do not add a
   second component or icon library.
4. The registry never goes stale: building a feature component and not registering it is
   a QA rejection.
