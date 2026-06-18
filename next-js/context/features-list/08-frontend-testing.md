# Feature 08 — Frontend Testing & Quality Gates

- Feature ID: FEAT-08
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01, 02, 03, 04, 05, 06, 07, 09

> Source of truth for the **frontend test suite + quality gates** — component tests,
> Playwright e2e on the critical flows, and accessibility checks, plus the wired
> scripts. `## Tasks` below is left for `/parse-features`.

## Goal

A trustworthy quality gate: component/unit tests for the feature slices and shared
libs, Playwright e2e for the critical user journeys, automated a11y checks, and the
existing lint/format/build gates — runnable locally and in CI.

## What exists today

### Tooling & config

- **Vitest** (jsdom) + **@testing-library/react** + **@testing-library/user-event** +
  **@testing-library/jest-dom**. Config in `vitest.config.ts` (globals, jsdom,
  `vite-tsconfig-paths` for the `@/*` alias, v8 coverage with text+html reporters).
- Shared test helpers in `src/test/`:
  - `render.tsx` — `renderWithProviders` wraps the unit under test in
    `NextIntlClientProvider` (real `en.json`) + a fresh `QueryClient` (retries off);
    re-exports RTL + `userEvent`.
  - `messages.ts` — imports the production `en.json` so test translations stay in
    lockstep with the app.
  - `setup.ts` — jest-dom matchers, DOM/storage cleanup per test, and a `PointerEvent`
    polyfill (jsdom lacks the constructor that base-ui's Checkbox dispatches).
- **Playwright** (`@playwright/test`) + `@axe-core/playwright`. Config in
  `playwright.config.ts`: chromium project, `webServer` runs `pnpm build && pnpm start`
  (override with `E2E_BASE_URL` to target a running app).
- Scripts in `package.json`: `test`, `test:watch`, `test:coverage`, `test:e2e`
  (alongside `lint`, `format`, `format:check`, `build`).
- **CI** (`.github/workflows/ci-checking.yml`) runs `pnpm lint`, `pnpm test`, and
  `pnpm build` on every change. (E2E is not yet wired into CI — see gaps.)

### Component / unit suites (Vitest — 14 files, 69 tests, green)

- **Typed client** (`shared/libs/api.test.ts`) — envelope success/failure, `ApiError`
  with `code`/`fields`, 204 handling.
- **Validators** — `identity/validators/auth.validator`, `message/validators`,
  `payment/validators`.
- **Auth service** (`identity/services/auth.service`) — token persist/clear, the 2FA
  pending-token hop, sign-out.
- **Zustand store** (`identity/store/auth.store`).
- **Representative client components — one+ per feature domain:**
  - identity: `sign-in-form` (validation, success redirect, 2FA no-redirect, pending),
    `sign-up-form` (validation, dto mapping + redirect, server `error.fields`, pending).
  - message: `message-list` (empty/rows), `message-compose-form`
    (channel validation, submit, sending state).
  - analytic: `stat-cards` (zeroed + per-type cards).
  - audit: `alert-table` (empty, rows, resolved label, resolve mutation, `error.code`
    via toast).
  - payment: `payment-list` (empty, rows, refund eligibility, refund amount, `error.code`).
  - notification: `notification-list` (empty/unread badge, mark-read, mark-all-read).
- Error assertions branch on `error.code` / `error.fields`, never on localized text.

### E2E suites (Playwright — 4 files, 6 tests, green)

- `landing.spec.ts` — marketing hero renders at `/en`; **axe** a11y (no
  critical/serious WCAG 2a/2aa violations).
- `route-guard.spec.ts` — unauthenticated `/en/dashboard` → `/en/sign-in`.
- `auth.spec.ts` — sign-in happy path (`/en/sign-in` → `/en/dashboard`, token
  persisted) and sign-out (dashboard → public `/sign-in`, token cleared). Backend
  envelopes are mocked with `page.route` (no live API).
- `locale.spec.ts` — locale switch `/en` → `/ru` persists across reload.

## Scope (reference)

- **Test runner / component tests:** Vitest + RTL (jsdom) with `test`/`test:watch`/
  `test:coverage` scripts. Covers typed client, validators, auth service (token
  persist + 2FA hop), store, and representative feature forms/tables.
- **E2E (Playwright):** config + `test:e2e` script. Critical flows: sign-in →
  dashboard, route-guard redirect, sign-out, locale switch. Backend mocked via
  `page.route` envelopes.
- **Accessibility:** axe checks on the public landing in Playwright.
- **Gates:** lint (`eslint`), format check (`prettier --check`), and `next build`
  stay green; coverage surfaced via `test:coverage`.
- **Out of scope:** backend test suite; visual-regression snapshots.

## Consumed API

- No new endpoints — tests exercise the same envelope contract via mocks/fixtures
  (Vitest mocks the query hooks; Playwright mocks `**/api/v1/**` with `page.route`).
  Assertions branch on `error.code`, never on i18n-localized message text.

## UI / components used

- Tests target existing feature `components/`, `services/`, `validators/`, and
  `shared/libs/*` — they add no product UI.

## Acceptance criteria

- [ ] `test` script runs the component/unit suite (Vitest + RTL) green; coverage
      reported via `test:coverage`.
- [ ] Typed client, validators, auth service (token persist + 2FA hop), and store are
      unit-tested.
- [ ] Representative feature forms/tables have component tests (one+ per domain)
      covering empty/error/success and pending states.
- [ ] Playwright `e2e` script runs the critical flows green (sign-in → dashboard,
      guard redirect, sign-out, locale switch).
- [ ] Automated a11y checks pass on the public landing.
- [ ] Tokens-only styling, lint, build, and tests pass.

## Remaining gaps (honest)

- **Coverage breadth:** overall statement coverage is low (~12%) because most
  presentational components and `app/` pages/layouts are untested; branch/function
  coverage on the exercised modules is higher (~73% / ~63%). The suite covers one
  representative client component per domain, not every component.
- **E2E breadth:** sign-up and a domain happy path (e.g. compose message) and a
  theme-toggle persistence flow are not yet covered by e2e (sign-up *is* covered at
  the component level).
- **a11y breadth:** axe runs on the landing only; auth/dashboard/list pages are not
  yet axe-scanned.
- **CI:** `test:e2e` is not wired into `ci-checking.yml` (it requires a built+started
  server and Playwright browsers); only lint/test/build run in CI today.
- **Server components / next-intl server APIs** are intentionally out of Vitest scope.

## Tasks

<!-- Left for /parse-features to decompose. -->
