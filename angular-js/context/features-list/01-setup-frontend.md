# Feature 01 — Frontend Setup & Scaffold

- Feature ID: FEAT-01
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: none

> Source of truth for the **frontend foundation** — the FEAT-01 of this Angular SPA.
> This project is **empty (no code exists yet)** — everything below is **TO BUILD**.
> `/parse-features 01` turns the `## Tasks` section below into the per-feature task
> file. This feature stands up a runnable, hardened Angular app shell; it writes
> **no auth or domain logic** beyond the wiring the shell needs. Identity pages land
> in FEAT-02.

## Goal

Stand up a production-grade Angular single-page web client (latest Angular,
standalone components, no NgModules) as a runnable foundation: routing + layout
shell, a typed `HttpClient` API service with the response envelope, auth/error
interceptors with JWT attach + refresh-retry, i18n, a token-driven design system
with light/dark theming, route guards, and a static Docker/nginx build — so the
team is **ready to write feature code**. Mirror the architecture and stack locked in
`context/architecture.md`, `context/ui-context.md`, and `context/api-context.md`.

## Scope

- **Angular CLI app** bootstrapped via `bootstrapApplication(AppComponent, appConfig)`
  — standalone components, `strict: true`, no `any`, no NgModules.
- **Routing** — `app.routes.ts` with lazy, standalone feature route configs;
  `app.config.ts` provides the router, `HttpClient`, and interceptors.
- **TS path aliases** (`tsconfig` paths): `@app/*`, `@core/*`, `@shared/*`,
  `@features/*`, `@env/*`.
- **Environment config** — `environments/environment.ts` + `environment.prod.ts`
  holding `apiBaseUrl` (`/api/v1`) and feature flags; no hardcoded URLs anywhere else.
- **Typed API service** (`core/http/api.service.ts`) — wraps `HttpClient`, unwraps the
  success/error/paginated envelope, exposes typed `get`/`post`/`patch`/`delete`.
  Envelope + domain types live in `core/models`.
- **Interceptors** (`core/interceptors/`):
  - `auth.interceptor` — attaches `Authorization: Bearer <accessToken>`; on `401`
    runs a single refresh-and-retry, else forces logout + redirect to login.
  - `error.interceptor` — maps HTTP failures (`400/401/403/404/409/429/5xx`) to typed
    client errors; never swallows errors.
- **Auth core** (`core/auth/`) — `auth.service` + token store using signals (access
  token in memory where possible, refresh per chosen strategy); tokens never logged.
  Full identity pages/flows are FEAT-02 — this provides the store + interceptor seam only.
- **Route guards** (`core/guards/`) — `auth.guard` and `role.guard` as `CanActivateFn`;
  unauthenticated → redirect to login; admin-only routes gated by role claims.
- **Layout shell** (`app/app.component.ts` + `layout/`) — app shell with
  `router-outlet`, header, optional sidebar; renders public vs authenticated chrome.
- **i18n** — runtime translation setup with a generic locale list (default `en`,
  plus at least one more), instant locale toggle, a typed translation service, and
  message catalogs under `shared/i18n/`.
- **Design tokens + theming** — tokens from `context/ui-context.md` defined once as
  CSS custom properties in `src/styles/`; light default + dark via `data-theme` on
  `<html>`; a theme toggle/service that persists the choice. If Tailwind is added,
  wire `@theme` to the same variables. No hardcoded visual values.
- **Shared design-system primitives** (`shared/components/`) — minimum: button, card,
  input, badge/chip — each with default/hover/focus-visible/disabled/loading states,
  registered in the `ui-context.md` component registry.
- **Quality tooling** — ESLint (`angular-eslint`) + Prettier, Husky + lint-staged;
  `ng test` (Jest or Karma) wired; Playwright wired for e2e (full suite is FEAT-08).
- **Docker/nginx build** — multi-stage `Dockerfile` (node build → `nginx:alpine`
  static serve), `nginx.conf` with SPA fallback (`try_files → /index.html`), and
  `docker/compose/docker-compose.{dev,staging,production}.yml`.
- Out of scope: any identity/domain pages, real backend calls beyond a health/probe
  smoke, the full e2e suite (FEAT-08), and the public marketing pages (FEAT-09).

## Consumed API

- Base URL from `environments/*` (`apiBaseUrl` → `/api/v1`); no host hardcoded in code.
- Response envelope mirrored in `core/models`:
  - success `{ success: true, data, meta? }`
  - error `{ success: false, error: { code, message, fields? } }`
  - paginated `meta: { page, limit, total, totalPages }`
- Auth seam: `Authorization: Bearer <accessToken>`; `401` → single refresh-and-retry
  in `auth.interceptor` (the concrete `/auth/refresh` call is finalized in FEAT-02).
- No feature endpoints are consumed here beyond an optional health probe.

## UI / components used

- Layout shell (header + optional sidebar) and a theme toggle.
- Shared primitives: Button, Card, Input, Badge/Chip — token-only styling, OnPush.
- Design tokens (colors, typography, spacing, radii, elevation, motion) per
  `context/ui-context.md`, defined as CSS custom properties with light/dark variants.

## Acceptance criteria

- [ ] App bootstraps via `bootstrapApplication` with `app.config.ts` providers
      (router, `HttpClient`, interceptors); no NgModules anywhere.
- [ ] TS path aliases (`@app/@core/@shared/@features/@env`) resolve; `strict: true`,
      no `any` in source.
- [ ] `environments/environment.ts` + `environment.prod.ts` provide `apiBaseUrl`; no
      hardcoded API host in components/services.
- [ ] Typed `api.service` unwraps the success/error/paginated envelope; components
      never call `HttpClient` directly.
- [ ] `auth.interceptor` attaches the bearer token and performs a single
      refresh-and-retry on `401`; `error.interceptor` maps failures to typed errors.
- [ ] `auth.guard` + `role.guard` (`CanActivateFn`) gate routes; unauthenticated →
      redirect to login; admin route gated by role.
- [ ] Layout shell renders with `router-outlet`; public vs authenticated chrome works.
- [ ] i18n configured with a default locale + at least one more, instant toggle, and
      message catalogs under `shared/i18n/`.
- [ ] Design tokens defined as CSS custom properties; light default + dark via
      `data-theme`; theme toggle persists; no hardcoded visual values.
- [ ] Shared primitives (button, card, input, badge/chip) implement
      default/hover/focus-visible/disabled/loading and are in the component registry.
- [ ] All components use `ChangeDetectionStrategy.OnPush`; state via signals/observables.
- [ ] Multi-stage `Dockerfile` + `nginx.conf` (SPA fallback) build a static image;
      `docker/compose/*` files present.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
