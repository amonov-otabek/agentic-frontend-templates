# Feature 08 — Frontend Testing & QA

- Feature ID: FEAT-08
- Status: implemented
- Depends on: FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-07, FEAT-09

> Source of truth for the **test harness**: unit/component tests (Vitest + Testing
> Library), Playwright end-to-end flows, and accessibility checks. This spec reflects what
> currently EXISTS in the repo — the installed tooling, configs, scripts, and the suites
> that run — plus the remaining gaps that were intentionally left out of scope.

## Goal

A working test suite for the SPA: Vitest + React Testing Library for component/unit tests
(with a mocked API client and mocked query hooks), Playwright for critical e2e user flows
against a `vite preview` build with intercepted backend responses, and automated a11y
assertions — wired to pnpm scripts.

## Test layout (HARD)

All tests in this feature follow `context/architecture.md` → **Test structure**: `test/` — `main.spec.ts(x)`, `app.spec.ts(x)`, `setup.ts`, `helpers.ts(x)`, folders mirroring `src/` (`*.spec.ts(x)`), `e2e/` (`*.e2e-spec.ts`, own `helpers.ts` allowed). Any path elsewhere in this spec that sits outside that layout (`src/**/tests/`, `src/test/`, a root `tests/` or `e2e/`, `*.test.*`, `conftest.py`) describes the current/legacy suite — it is migrated by this feature, never extended.

## What exists today

### Tooling & config (installed)

- `vitest` (`jsdom` env) + `@vitest/coverage-v8`.
- `@testing-library/react`, `@testing-library/user-event`, `@testing-library/jest-dom`.
- `@playwright/test` + `@axe-core/playwright`.
- `vitest.config.ts` / Vitest config (jsdom, `src/test/setup.ts`), `playwright.config.ts`
  (chromium project, `webServer` runs `pnpm build && pnpm preview` on port 4173).
- `src/test/setup.ts` — registers `@testing-library/jest-dom`, clears storage between tests,
  polyfills `matchMedia`, and polyfills `PointerEvent` + `*PointerCapture` so Radix
  primitives (checkbox, etc.) work under user-event in jsdom.
- `src/test/render.tsx` — `renderWithProviders` helper mounting components with i18n,
  React Query (fresh client, retries off), and an in-memory router.

### Scripts (`package.json`)

`test` (Vitest run), `test:watch`, `test:coverage`, `test:e2e` (Playwright),
`test:e2e:ui`.

### API mocking

- Component tests mock the feature query hooks (`vi.mock("../queries/*.queries")`) and the
  toast hook, so they assert on rendered envelope-shaped data and on `ApiError.fields`
  without a live backend.
- `shared/libs/api.ts` is unit-tested directly against a mocked `fetch`.
- E2E mocks the backend with Playwright `page.route(...)` returning the
  `{ success, data }` envelope (see `e2e/helpers/api-mock.ts`).

### Unit / component tests (Vitest — 13 files, 62 tests, all green)

- `shared/libs/api.test.ts` — envelope unwrap, typed `ApiError`, 204, pagination `meta`.
- `identity/services/auth.service.test.ts` — token persist/clear, refresh, pending-2FA.
- Zod validators: `identity/validators/auth.validator.test.ts`,
  `message/validators/message.validator.test.ts`,
  `payment/validators/payment.validator.test.ts`.
- One+ representative component per feature domain:
  - identity — `sign-in-form.test.tsx`, `sign-up-form.test.tsx` (validation, submit DTO,
    `ApiError.fields` surfacing).
  - message — `message-list.test.tsx`, `message-compose-form.test.tsx`.
  - analytic — `stat-cards.test.tsx` (empty/zero + card-per-type).
  - audit — `alert-table.test.tsx` (empty, rows, resolve action, resolved state).
  - payment — `payment-list.test.tsx` (empty, rows, refund remaining-amount, hidden when
    not refundable).
  - notification — `notification-list.test.tsx` (empty, rows + unread badge, mark-read,
    mark-all-read).

### E2E (Playwright — 4 files, 8 tests, all green, chromium)

- `e2e/landing.spec.ts` — marketing hero/CTAs + axe (no serious/critical violations).
- `e2e/routeGuard.spec.ts` — unauthenticated `/dashboard` → `/sign-in`; public route stays
  reachable.
- `e2e/authFlow.spec.ts` — sign-in happy path lands on `/dashboard` (asserts stored access
  token); sign-out returns to `/sign-in` and clears the token. Backend mocked via
  `page.route`.
- `e2e/preferences.spec.ts` — locale switch persists across reload (`i18nextLng`); theme
  toggle persists across reload (`theme`).

## Remaining gaps (not yet done)

- Coverage is wired (`test:coverage`) but the ≥80% threshold is **not enforced** and the
  numbers were not measured/asserted as part of this work.
- Unit tests do not yet cover `usePagination` / `usePermission` hooks.
- E2E does not include a dedicated read-flow assertion per dashboard feature (dashboard is
  reached via the auth flow, but per-feature data reads are not separately asserted).
- Axe assertions run on the public landing page only; sign-in / dashboard / settings axe
  passes are not yet added.
- CI wiring for `test` / `test:coverage` / `test:e2e` is not part of this change.
- Out of scope (by design): visual-regression/screenshot diffing, load/perf testing.

## Consumed API

- None live. Component tests mock query hooks + `shared/libs/api.ts`; e2e intercepts the
  `{ success, data, meta? }` envelope via Playwright `page.route`.

## UI / components used

- Test utilities under `src/test/*` (`renderWithProviders` wrapping i18n / Query / router);
  e2e helpers under `e2e/helpers/*`; existing feature components and pages as units under
  test.

## Acceptance criteria

- [ ] Test layout matches `context/architecture.md` → Test structure: every existing test is moved to its mirrored path and renamed to the stack suffix; nothing test-related remains under the source root; runner config (globs, setup file, e2e dir, coverage root) points at the test folder; the collected-test count is not lower after the move.
- [x] Vitest (jsdom) + Testing Library + Playwright + axe are installed and configured;
      `test`, `test:watch`, `test:coverage`, `test:e2e`, `test:e2e:ui` scripts run.
- [x] `renderWithProviders` (i18n + Query + router) and `src/test/setup.ts` exist and back the suite.
- [x] `shared/libs/api.ts` is unit-tested (envelope unwrap, typed `ApiError`, 204, `meta`)
      and `auth.service` is tested (persist/clear, refresh, pending-2FA).
- [x] At least one component test per feature domain runs green against mocked query hooks / client.
- [x] Playwright covers sign-in → dashboard, route-guard redirect, sign-out, locale + theme
      persistence, with the backend mocked via `page.route`; axe passes on the landing page.
- [x] Component tests assert on `error.code` / `ApiError.fields`, never localized message text.
- [ ] ≥80% coverage threshold enforced and measured (remaining gap).
- [ ] `usePagination` / `usePermission` unit tests, per-feature read-flow e2e, axe on
      sign-in / dashboard / settings, and CI wiring (remaining gaps).

## Tasks

- [x] Install + configure Vitest, Testing Library, Playwright, axe; add test scripts.
- [x] `renderWithProviders` helper (i18n + Query + router) and `src/test/setup.ts`.
- [x] Unit-test `shared/libs/api.ts` (envelope, `ApiError`, 204, `meta`) and `auth.service`.
- [x] At least one component test per feature (identity, analytic, audit, payment,
      notification, message) against mocked hooks/client.
- [x] Playwright: sign-in → dashboard, unauthenticated route-guard redirect, sign-out,
      locale switch, theme toggle (backend mocked via `page.route`).
- [x] Axe on the public landing page (no serious/critical violations).
- [ ] Enforce ≥80% coverage threshold and measure it.
- [ ] Unit tests for `usePagination` / `usePermission`.
- [ ] Per-feature read-flow e2e + axe on sign-in / dashboard / settings.
- [ ] Wire tests + coverage + e2e into CI alongside the build gate.
