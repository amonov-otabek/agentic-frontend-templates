# UI Context — Design System

Machine-readable design system: tokens, theme, and the living component registry.
`context/ui-rules.md` says how to use these values; this file IS the values. This is
the planned foundation for the Angular SPA — it is locked before the first UI task is
built, because retrofitting a design system is how features get rebuilt.

## Theme

- Mode: **light default, dark supported** (toggle via a `data-theme` attribute on
  `<html>`; tokens below redefine under `[data-theme="dark"]`).
- Personality: clean, professional, content-first. Calm neutrals, one decisive accent,
  generous whitespace, restrained motion.
- Reference designs: _add paths/links here when available (e.g. `designs/*.png`, Figma URL)._

## Tokens

Single source of truth. Defined once as **CSS custom properties** on `:root` in
`src/styles/` and referenced everywhere by name — components never restate values.
If Tailwind is added, wire its `@theme` to these same variables (one source, two faces).

```css
:root {
  /* Colors — light */
  --color-background: #ffffff;
  --color-surface: #f8fafc;
  --color-foreground: #0f172a;
  --color-muted: #f1f5f9;
  --color-muted-foreground: #64748b;
  --color-border: #e2e8f0;
  --color-accent: #2563eb;
  --color-accent-foreground: #ffffff;
  --color-accent-hover: #1d4ed8;
  --color-success: #16a34a;
  --color-warning: #d97706;
  --color-destructive: #dc2626;
  --color-ring: #93c5fd;            /* focus ring */

  /* Typography */
  --font-sans: "Inter", system-ui, -apple-system, "Segoe UI", sans-serif;
  --font-mono: "JetBrains Mono", ui-monospace, monospace;
  --text-xs: 12px;
  --text-sm: 14px;
  --text-base: 16px;
  --text-lg: 18px;
  --text-xl: 24px;
  --text-2xl: 32px;
  --leading-tight: 1.25;
  --leading-normal: 1.5;
  --weight-regular: 400;
  --weight-medium: 500;
  --weight-semibold: 600;
  --weight-bold: 700;

  /* Radii */
  --radius-sm: 6px;
  --radius-md: 10px;
  --radius-lg: 16px;
  --radius-full: 9999px;

  /* Spacing scale (4-based) */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;

  /* Elevation */
  --shadow-sm: 0 1px 2px rgba(15, 23, 42, 0.06);
  --shadow-md: 0 4px 12px rgba(15, 23, 42, 0.10);
  --shadow-lg: 0 12px 32px rgba(15, 23, 42, 0.14);

  /* Motion */
  --duration-fast: 120ms;
  --duration-base: 200ms;
  --easing-standard: cubic-bezier(0.2, 0, 0, 1);
}

[data-theme="dark"] {
  --color-background: #0f172a;
  --color-surface: #1e293b;
  --color-foreground: #f8fafc;
  --color-muted: #1e293b;
  --color-muted-foreground: #94a3b8;
  --color-border: #334155;
  --color-accent: #3b82f6;
  --color-accent-foreground: #0b1220;
  --color-accent-hover: #60a5fa;
  --color-ring: #1d4ed8;
}
```

### Color usage guide

| Use | Token |
| --- | --- |
| Page background | `--color-background` |
| Card / raised surface | `--color-surface` |
| Primary text | `--color-foreground` |
| Secondary / metadata text | `--color-muted-foreground` |
| Primary actions / active nav | `--color-accent` (hover `--color-accent-hover`) |
| Text on accent | `--color-accent-foreground` |
| Borders / dividers | `--color-border` |
| Focus ring | `--color-ring` |
| Success / warning / error states | `--color-success` / `--color-warning` / `--color-destructive` |

## Typography scale

| Style | Token (size / weight / leading) | Use |
| --- | --- | --- |
| Display | `--text-2xl` / `--weight-bold` / `--leading-tight` | hero, landing |
| Heading 1 | `--text-xl` / `--weight-bold` / `--leading-tight` | page titles |
| Heading 2 | `--text-lg` / `--weight-semibold` / `--leading-tight` | section titles |
| Body | `--text-base` / `--weight-regular` / `--leading-normal` | default text |
| Small | `--text-sm` / `--weight-regular` / `--leading-normal` | dense UI, labels |
| Caption | `--text-xs` / `--weight-regular` / `--leading-normal` | metadata, hints |

Base font is `--font-sans`, loaded once and exposed as a token. Base size ≥ 16px on
mobile (no zoom traps).

## Spacing

4-based scale: `--space-1`(4) / `2`(8) / `3`(12) / `4`(16) / `6`(24) / `8`(32) /
`12`(48) / `16`(64). Conventions: card padding `--space-6` (24), section vertical gaps
`--space-8` (32), form field gaps `--space-4` (16), inline control gaps `--space-2` (8).
Max content width 1280px, centered, horizontal padding `--space-6`.

## Component registry (living document)

Updated by Builders after every new component; checked by Builders before every new
component; audited by QA and /review. One entry per component:

```markdown
### ComponentName
- Path: `app/shared/components/<name>/<name>.component.ts` (or `app/features/<feature>/components/...`)
- Purpose: one line
- Inputs/outputs: the public API surface (`input()` / `output()`)
- States: interaction (default/hover/focus-visible/disabled/loading) + data (loading/empty/error/success) where applicable
- Used by: pages/features using it
```

_(empty — first entry added by the first UI task)_

## Invariants

1. No visual value outside this file — every color, size, radius, and spacing is a token.
2. New tokens are added here first, then used — never inline-then-backfill.
3. The registry never goes stale: building a component and not registering it is a QA
   rejection.
4. Every interactive component implements default / hover / focus-visible / disabled /
   loading; every async surface implements loading / empty / error / success.
