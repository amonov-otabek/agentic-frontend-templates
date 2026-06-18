# Feature 09 — Public Marketing Site (Frontend)

- Feature ID: FEAT-09
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01

> Source of truth for the public-facing **marketing landing** under the `(public)`
> route group — no auth, SEO-optimized, localized. Its CTAs route visitors into the
> `(auth)` and `(protected)` areas. `## Tasks` below is left for `/parse-features`.

## Goal

A fast, SEO-friendly, localized public landing page (and any supporting marketing /
legal pages) on the `(public)` route group that introduces the product and converts
visitors into sign-ups — fully public, statically rendered where possible.

## Scope

- **Route group:** `src/app/[locale]/(public)/` with its own `layout.tsx` wrapping
  pages in the marketing shell (header + footer), separate from the dashboard shell.
- **Landing** (`(public)/page.tsx`): hero (tagline / title / subtitle), primary CTAs
  (`get-started` → `(auth)` sign-up, `learn-more`), value-prop / how-it-works
  sections, footer. Expand the existing minimal landing into the marketing page(s).
- **Marketing shell:** `src/layout/public-header.tsx` (logo, locale switch, theme
  toggle, sign-in / sign-up CTAs) + `src/layout/public-footer.tsx` (links, socials).
- **i18n:** all copy through `next-intl` under the `marketing.*` / `common.*` keys in
  `messages/{en,ru,uz}.json`; locale-prefixed routes; instant locale toggle.
- **SEO / metadata:** per-page `<title>` / meta description / canonical via App
  Router `metadata`/`generateMetadata`, Open Graph + Twitter cards, JSON-LD
  (Organization) on the landing, `sitemap.ts`, `robots.ts`, per-locale `hreflang`;
  statically generated (SSG) where possible.
- **404 / not-found** localized handling.
- **Out of scope:** authenticated product screens (those are `(protected)`); auth
  forms (FEAT-02). No backend calls beyond an optional contact form (flag the target
  if added).

## Consumed API

- None required for the landing (public, static). If a contact form is added, it
  posts through the typed client (`shared/libs/api.ts`) to a backend
  notification/message endpoint under `${NEXT_PUBLIC_API_URL}/api/v1` — flag the
  chosen path; otherwise the page is API-free.

## UI / components used

- `src/app/[locale]/(public)/{layout,page}.tsx`; `src/layout/{public-header,public-footer}.tsx`;
  shared widgets `locale-switch`, `theme-toggle`, `icons`; `button`/`buttonVariants`,
  `card`, `accordion` (FAQ), `carousel` (hero) from `shared/components/ui/*`;
  `Link` from `shared/i18n/navigation`.

## Acceptance criteria

- [ ] `(public)` landing renders in the marketing shell (header + footer), no auth
      required, responsive desktop → mobile.
- [ ] CTAs deep-link into `(auth)` sign-up / sign-in via locale-aware `Link`.
- [ ] All copy localized in `en`/`ru`/`uz` with `localePrefix: 'always'`; locale
      toggle + theme toggle work.
- [ ] SEO: per-page metadata, OG / Twitter cards, JSON-LD Organization on the
      landing, `sitemap.xml`, `robots.txt`, `hreflang`; pages statically generated.
- [ ] Localized 404 / not-found handled.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
