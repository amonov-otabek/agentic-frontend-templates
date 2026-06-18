# Feature 09 — Public Site (Frontend)

- Feature ID: FEAT-09
- Status: draft
- Depends on: FEAT-01 (scaffold: public layout, theme, i18n, router)

> Source of truth for the **public, no-auth** marketing surface rendered under
> `PublicLayout` at `/`. It is the top-of-funnel landing area whose CTAs route visitors
> into the auth flow (`/sign-up`, `/sign-in`); it consumes no authenticated API.

## Goal

Build the public landing page(s) on `/`: a fast, localized, themed marketing surface with
a hero, value proposition, and clear CTAs into sign-up / dashboard, plus the SPA-appropriate
meta/SEO baseline and a not-found page.

## Scope

- **Home page** (`src/app/home-page`, route `/` under `PublicLayout`) — hero (tagline,
  title, subtitle), primary CTA → `/sign-up`, secondary CTA → `/dashboard` (or `/sign-in`),
  responsive down to mobile.
- **Public shell** (`src/layout/{public-layout,public-header,public-footer}`) — header
  (logo, nav, locale switch, theme toggle, sign-in CTA) + footer (links, copyright)
  wrapping all public routes.
- **Optional public routes** — extend the `PublicLayout` group with simple static pages as
  needed (e.g. `/about`, legal) built from the same shell; keep them no-auth and localized.
- **Not-found** — a 404 route/element for unmatched paths, themed and localized.
- **Meta / SEO (SPA-appropriate)** — document `<title>` + meta description per public
  route (e.g. via a small head helper / `react-helmet`-style or direct `document` updates),
  favicon, Open Graph defaults, and `robots.txt`. Note: this is a static client SPA (no
  SSR) — content is client-rendered; deep SEO/prerender is the `next-js` template's job.
- All copy through i18n (en/ru/uz); all visual values from tokens; light/dark via `next-themes`.
- Out of scope: server-side rendering / prerendering, blog/CMS, authenticated content,
  a backend-backed contact form (none in the consumed API by default).

## Consumed API

- None. The public site is no-auth and makes no `/api/v1` calls; CTAs are client-side
  navigations into the auth flow.

## UI / components used

- `src/app/home-page`, `src/layout/{public-layout,public-header,public-footer}`.
- shadcn `button` (`buttonVariants`), `card`; widgets `icons`, `locale-switch`,
  `theme-toggle`, `sticky-header`; `react-router-dom` `Link`; tokens in `globals.css`.

## Acceptance criteria

- [ ] `/` renders under `PublicLayout` with a hero and CTAs that navigate to `/sign-up`
      and `/dashboard` (or `/sign-in`); no auth required.
- [ ] The public header (logo, nav, locale switch, theme toggle, sign-in CTA) and footer
      wrap all public routes and are responsive (desktop → mobile).
- [ ] A 404/not-found route handles unmatched paths, themed and localized.
- [ ] Per-route `<title>` + meta description set; favicon, OG defaults, and `robots.txt`
      present.
- [ ] All copy localized (en/ru/uz) with an instant locale toggle; light/dark theme works.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
