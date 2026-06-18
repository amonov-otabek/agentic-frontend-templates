# Feature 09 — Public Site (Frontend)

- Feature ID: FEAT-09
- Status: draft
- Depends on: FEAT-01 (scaffold: PublicLayout, router, i18n, theme)

> Source of truth for the **public, no-auth** marketing/landing surface served by
> the SPA. `/parse-features 09` turns the `## Tasks` section into
> `context/tasks-list/09-public-site-tasks.md`. This is the top-of-funnel surface
> whose CTAs route visitors into the auth flow (FEAT-02).

## Goal

Build the anonymous-visitor landing surface under `PublicLayout`: a marketing
home page (and any supporting public routes) introducing the product, with the
shared public header/footer, locale switcher, theme toggle, sign-in/sign-up CTAs,
and per-route document metadata — themed and localized, no auth required.

## Scope (real pages, routes, components)

- **Home** (`src/app/HomeView.vue`): mounted at `/` (route name `home`) under
  `PublicLayout` — hero / value prop / feature highlights / CTA into `/sign-up`
  and `/sign-in`.
- **Supporting public routes** (added under the `PublicLayout` children in
  `src/app/router.ts`, no `requiresAuth`): e.g. `/about`, `/contact`, `/privacy`,
  `/terms` — generic marketing/legal pages built from shared public components.
- **Shared shell** (`src/layout/`): `public-layout.vue` wraps `public-header.vue`
  (logo, nav, `locale-switch`, `theme-toggle`, primary CTA) + `public-footer.vue`
  (legal/links, socials).
- **Components**: reuse `shared/components/ui/*` (button, card, accordion for FAQ)
  and `widgets` (locale-switch, theme-toggle, sticky-header).
- **Metadata/SEO (SPA-appropriate):** per-route `document.title` + meta
  description via a `beforeEach`/composable using route `meta.titleKey`; static
  `index.html` defaults (title, description, Open Graph, favicon); `robots.txt`
  in `public/`. Note: this is a static SPA (no SSR), so SEO is best-effort
  client-side + prerendered `index.html` defaults.
- Out of scope: SSR/SSG prerendering, blog/CMS, and dynamic sitemaps (flag if
  scoped; the template is static-served by nginx).

## Consumed API

- None required for marketing pages. An optional contact form may POST to a
  backend message/notification endpoint (`/api/v1/messages`) if scoped — flag the
  chosen target; otherwise the form is presentational/disabled.

## UI / components used

- `public-layout`, `public-header`, `public-footer`; shadcn-vue `button`, `card`,
  `accordion`, `navigation-menu`; `locale-switch`, `theme-toggle` widgets.

## Acceptance criteria

- [ ] `/` renders the public home under `PublicLayout` with no auth required; CTAs route to `/sign-in` and `/sign-up`.
- [ ] Public header + footer render across all public routes with logo, nav, locale switcher, theme toggle, and primary CTA.
- [ ] Supporting public routes (about/contact/legal) render under `PublicLayout` without `requiresAuth`.
- [ ] Per-route `document.title`/meta updates on navigation; `index.html` ships sensible title/description/OG defaults; `robots.txt` present.
- [ ] All marketing/legal copy localized in en/ru/uz with instant locale toggle; light/dark both render correctly.
- [ ] Responsive from desktop down to mobile breakpoints.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
