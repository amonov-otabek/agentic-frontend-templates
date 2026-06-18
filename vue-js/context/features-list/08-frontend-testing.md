# Feature 08 — Frontend Testing & Quality Gates

- Feature ID: FEAT-08
- Status: draft
- Depends on: FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-07, FEAT-09 (every surface that gets tested)

> Source of truth for the **frontend test suite + quality gates**.
> `/parse-features 08` turns the `## Tasks` section into
> `context/tasks-list/08-frontend-testing-tasks.md`. Adds the testing toolchain
> the template ships without, then covers the built surfaces.

## Goal

Stand up the test toolchain (Vitest + @vue/test-utils + @testing-library/vue +
Playwright + axe) and cover the SPA: unit/component tests for forms, queries,
stores, the API client, and feature components; Playwright e2e for the critical
flows; and automated a11y checks — all wired into runnable scripts and CI.

## What exists (real toolchain, scripts, suites)

### Toolchain & config (installed, working)

- **Runner:** `vitest` + `@vitejs/plugin-vue` with `@vue/test-utils`,
  `@testing-library/vue`, `@testing-library/jest-dom`.
- **E2E:** `@playwright/test` + `@axe-core/playwright`.
- **`vitest.config.ts`** — jsdom env, `globals: true`, `@`-alias resolution,
  `setupFiles: ['./src/test/setup.ts']`, `css: false`, v8 coverage with
  `include: src/**/*.{ts,vue}` and excludes for tests/types/`main.ts`. Kept
  separate from `vite.config.ts` so the Tailwind Vite plugin is not loaded under test.
- **`playwright.config.ts`** — `testDir: ./e2e`, chromium project, auto-starts a
  `pnpm build && pnpm preview` web server on :4173; network is route-intercepted
  inside specs, so no live backend is required.
- **`src/test/setup.ts`** — registers jest-dom matchers and exports the shared
  mount harness: `createTestPlugins()` (fresh Pinia + memory-history Router +
  isolated `QueryClient` with retries off) and `withPlugins()` (adds real `en`
  i18n so assertions match rendered copy). Transition/teleport are stubbed globally.

### Scripts (`package.json`, all wired)

- `test` (`vitest run`), `test:watch` (`vitest`), `test:coverage` (`vitest run --coverage`),
  `e2e` / `test:e2e` (`playwright test`), `e2e:ui` (`playwright test --ui`).

### Unit / component suite (13 files, 60 tests — green)

- **Shared:** `shared/libs/api.test.ts` (envelope unwrap, `ApiError`,
  success/failure/204/field errors, base URL from `VITE_API_URL`),
  `shared/libs/validators.test.ts` (`formatZodErrors`).
- **identity:** `auth.store.test.ts` (status transitions), `auth.service.test.ts`
  (token persist/clear + pending-2FA), `auth.validator.test.ts` (Zod),
  `sign-in-form.test.ts` and `sign-up-form.test.ts` (render + validate + submit +
  server field-error surfacing via `ApiError.fields`).
- **message:** `message-compose-form.test.ts` (channel→recipient mapping + submit),
  `message.validator.test.ts`.
- **audit:** `audit-log-table.test.ts` (empty state + rows).
- **analytic:** `stat-cards.test.ts` (total + per-type cards from props).
- **payment:** `payment-list.test.ts` (rows, refund-eligibility, refund mutate
  called with outstanding amount).
- **notification:** `notification-list.test.ts` (empty/unread rendering,
  per-item mark-read + mark-all-read mutations).
- **Pattern:** feature `queries` modules and `@/shared/hooks/use-toast` are mocked
  with `vi.hoisted` spies + a real `ref` for `isPending`; component tests mount via
  the plugins helper and assert on rendered copy / `error.code` + `error.fields`,
  not message text.

### E2E suite (4 files, 7 tests — green, network mocked)

- `landing.spec.ts` — marketing hero renders; axe reports zero serious/critical
  violations (wcag2a/wcag2aa).
- `guard.spec.ts` — unauthenticated `/dashboard` → `/sign-in`; sign-in form renders.
- `auth-flow.spec.ts` — sign-in happy path (mocked `POST /auth/sign-in` envelope) →
  `/dashboard` with token persisted; sign-out → `/sign-in` with session cleared.
- `theme.spec.ts` — theme toggle persists to `localStorage('ui-theme')` and
  re-applies the `dark` class after reload.

### CI

- `.github/workflows/ci-checking.yml` runs on push/PR: `pnpm build` (`vue-tsc -b`
  typecheck + `vite build`), `pnpm test`, Playwright browser install, then `pnpm e2e`.

## Consumed API

- None live — the API client is mocked in unit/component tests and route-intercepted
  in e2e (envelope shape per `shared/libs/api.ts`).

## UI / components used

- All feature pages/components as test subjects; no new UI.

## Status vs. acceptance criteria

- [x] Vitest + Testing Library + Playwright + axe installed and configured with `@`-alias resolution and a shared test setup.
- [x] `test`, `test:coverage`, and `e2e` scripts run green locally.
- [~] Coverage: branch ~83–86% and function ~72–84% on exercised code, but the v8
  `include` spans the full vendored shadcn UI library (~339 primitives) alongside
  38 feature components, so the **global statement gate of ≥80% is not met** (~12%
  overall / ~26% over features+libs). Closing it cleanly is a config decision
  (narrow the coverage `include` to app code, or test the UI primitives) and was
  left out of scope here since the task was add-tests-only. **Remaining gap.**
- [x] Component tests cover one representative per feature domain (identity sign-in
  + sign-up, message compose, audit log table, analytic stat-cards, payment list +
  refund, notification list + mark-read); API client, store, service, and validator
  units covered; assertions use `error.code` / `error.fields`, not message text.
  - Not every form/table is covered yet (e.g. `users-table`, `invoice-list`,
    `preferences-form`, `forgot`/`reset` forms, query hooks) — **remaining gap.**
- [x] Playwright e2e covers sign-in happy path, guard redirect, sign-out, and theme
  persistence, with mocked network. (sign-up→verify, forgot→reset, locale switch,
  and a per-feature domain flow each remain — **remaining gap.**)
- [x] axe reports zero serious/critical violations on the landing page.
  - Dashboard/auth-page axe sweeps not yet added — **remaining gap.**
- [x] CI workflow runs build (typecheck) + test + e2e on push/PR. (Lint is not yet
  a CI step — **remaining gap.**)
- [x] Tokens-only (tests add no styling); lint (0 errors), typecheck, build, unit,
  and e2e all pass locally.
- Out of scope: visual-regression snapshots and load testing.

## Tasks
