# Feature 09 — Public Marketing Site (Frontend)

- Feature ID: FEAT-09
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01 (foundation: routing, layout shell, theme, i18n)

> Source of truth for the **public, unauthenticated** marketing surface of the Angular
> SPA. This project is **empty (no code exists yet)** — everything below is **TO
> BUILD**. `/parse-features 09` turns the `## Tasks` section into the per-feature task
> file. These pages require **no auth**; their CTAs route visitors into the app's
> sign-up / sign-in.

## Goal

Build the public **landing + legal** pages of the Angular SPA: a marketing home page
with a shared public header/footer shell, supporting legal/help pages, locale
switching, SEO/meta + a 404 — fast, accessible, themed, and localized. CTAs deep-link
into the authenticated app (FEAT-02 sign-up/sign-in).

## Scope

### Routes / pages (`features/public/`)

A lazy `public.routes.ts` under a `(public)` group (no `auth.guard`):

| Page | Route | Purpose |
| --- | --- | --- |
| Home | `/` | hero / value prop / how-it-works / features / CTAs / footer |
| About | `/about` | company / mission |
| Contact | `/contact` | contact details + contact form |
| Help / FAQ | `/help` | help center / FAQ |
| Privacy | `/privacy` | legal |
| Terms | `/terms` | legal |
| 404 | wildcard | not-found page |

### Components (`features/public/components/`)

`public-header` (logo, nav, locale switcher, theme toggle, primary CTA) ·
`public-footer` (legal/help links, socials) · `hero` · `feature-grid` ·
`how-it-works` · `cta-band` · `contact-form` (Reactive Forms + typed validators,
loading/success/error states) · `faq-accordion` · `legal-prose` (shared layout for
legal pages with TOC/anchors) · `not-found-view`.

### Services / state

- A small `public.service` (only if the contact form posts to the backend); otherwise
  pages are static-data + i18n driven. No protected data, no tokens.
- SEO/meta set per route via Angular's `Title` + `Meta` services (per-page title,
  description, canonical, Open Graph + Twitter cards, JSON-LD on home, per-locale
  `hreflang`).

### Guards

- **None** — public group is explicitly outside `auth.guard`. Authenticated users may
  still view it; CTAs route to `/auth/sign-up` (FEAT-02).

## Consumed API

- The site is primarily static. **Only** the contact form may call the backend:
  - `POST /messages` (the backend `message` domain) — if contact submissions are sent
    as messages; otherwise the form posts to a chosen relay/endpoint (flag the choice).
- If it posts to the API, register the endpoint in `context/api-context.md` and ensure
  the app origin is allowed by backend CORS.

## UI / components used

- Shared primitives (button, card, input) + the new public components and the public
  shell; responsive down to mobile breakpoints.
- Theme tokens only (light default + dark); every async surface (contact form)
  implements loading / success / error; OnPush throughout; all copy localized.

## Acceptance criteria

- [ ] Public route group renders with a shared header (logo, nav, locale switcher,
      theme toggle, primary CTA) + footer, outside `auth.guard`.
- [ ] Home, About, Help/FAQ, Privacy, Terms pages render, responsive (desktop →
      mobile), and localized in the configured locales.
- [ ] CTAs deep-link into `/auth/sign-up` / `/auth/sign-in` (FEAT-02).
- [ ] Contact form validates via Reactive Forms + typed validators with
      loading/success/error states; submission target chosen and flagged (and the
      endpoint registered in `api-context.md` if it hits the API).
- [ ] SEO: per-page title/description/canonical, Open Graph + Twitter cards, JSON-LD
      on home, per-locale `hreflang`, and a 404/not-found page.
- [ ] Legal pages use a shared prose layout with anchors/TOC where appropriate.
- [ ] Theme light/dark toggle works on the public shell; instant locale toggle works.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
