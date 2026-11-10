# Feature 08 — Frontend Testing & Quality Gate

- Feature ID: FEAT-08
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01, FEAT-02, FEAT-03, FEAT-04, FEAT-05, FEAT-06, FEAT-07, FEAT-09
  (all feature surfaces must exist to be tested)

> Source of truth for the **test suite + quality gate** across the Angular SPA. This
> project is **empty (no code exists yet)** — everything below is **TO BUILD**
> (incrementally as features land, then consolidated here). `/parse-features 08` turns
> the `## Tasks` section into the per-feature task file.

## Goal

Establish a comprehensive, repeatable test + quality gate for the Angular SPA:
component/unit tests (`ng test`), end-to-end tests (Playwright) for the critical user
flows, and automated accessibility checks — so every feature ships verified and
regressions are caught before merge.

## Test layout (HARD)

All tests in this feature follow `context/architecture.md` → **Test structure**: `test/` — `main.spec.ts(x)`, `app.spec.ts(x)`, `setup.ts`, `helpers.ts(x)`, folders mirroring `src/` (`*.spec.ts(x)`), `e2e/` (`*.e2e-spec.ts`, own `helpers.ts` allowed). Any path elsewhere in this spec that sits outside that layout (`src/**/tests/`, `src/test/`, a root `tests/` or `e2e/`, `*.test.*`, `conftest.py`) describes the current/legacy suite — it is migrated by this feature, never extended.

## Scope

### Component / unit tests (`ng test`)

- Specs in root `test/`, mirroring `src/` (`test/app/features/<name>/…`, `test/app/core/…`,
  `test/app/shared/…`); generate with `--skip-tests` so the CLI never colocates a spec.
- **Core:** `api.service` (envelope unwrap, error mapping), `auth.interceptor`
  (bearer attach + `401` refresh-retry), `error.interceptor`, `auth.guard` +
  `role.guard`, `auth.service` token store.
- **Per feature:** smart-page state logic + each presentational component's state
  matrix (default/hover/focus-visible/disabled/loading; loading/empty/error/success).
- HTTP mocked via `HttpTestingController`; signals/observables asserted; no real network.
- Coverage target met across statements/branches/functions/lines.

### E2E tests (Playwright)

Critical flows against the running app (mocked or test backend):

- Identity: sign-up → verify → sign-in → reach a protected route → sign-out; refresh
  rotation; guard redirect for unauthenticated access; admin-route role gate.
- Analytics dashboard loads with data, empty, and error states.
- Audit log filter + alert resolve.
- Payments list → detail → invoice download.
- Notifications: feed, mark-all-read, bell unread badge, preferences save.
- Messages: compose → send → appears in list with status.
- Public site: landing loads, locale toggle, primary CTA navigates.
- Theme: light/dark toggle persists across reload.

### Accessibility checks

- Automated a11y assertions (e.g. axe) in Playwright on key pages: focus order,
  visible focus, contrast, semantic landmarks/headings, alt text, keyboard reachability.

### CI quality gate

- A single command path runs lint + format check + typecheck + `ng test` (coverage) +
  `ng build` + Playwright e2e; failures block. Artifacts (coverage, traces,
  screenshots, videos) collected; flaky e2e quarantined, not deleted.

## Consumed API

- Tests mock the `/api/v1` envelope (success/error/paginated) via
  `HttpTestingController` (unit) and route mocks or a test backend (e2e). No new
  endpoints consumed; tests assert the existing contracts and branch on `error.code`.

## UI / components used

- Exercises every feature's components and the shared design-system primitives;
  asserts each component's documented state matrix from the `ui-context.md` registry.

## Acceptance criteria

- [ ] Test layout matches `context/architecture.md` → Test structure: every existing test is moved to its mirrored path and renamed to the stack suffix; nothing test-related remains under the source root; runner config (globs, setup file, e2e dir, coverage root) points at the test folder; the collected-test count is not lower after the move.
- [ ] Core (`api.service`, both interceptors, both guards, `auth.service`) unit-tested,
      including envelope unwrap, `401` refresh-retry, and error mapping.
- [ ] Each feature has component/page specs covering its state matrix; HTTP mocked via
      `HttpTestingController`; no real network in unit tests.
- [ ] Coverage target met and enforced (statements/branches/functions/lines).
- [ ] Playwright e2e covers all critical flows listed above against a mocked/test
      backend; tests are deterministic (flaky ones quarantined).
- [ ] Automated a11y checks pass on key pages (focus, contrast, landmarks, keyboard).
- [ ] One gate command runs lint + format + typecheck + `ng test` + `ng build` + e2e;
      artifacts collected; the gate blocks on failure.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
