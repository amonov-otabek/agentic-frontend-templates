# Project Standards

Implementation rules for all code. Planned foundation for the Angular SPA client.
Treat as locked once the first feature is built.

## Engineering mindset

1. **Think first** — read the task block and relevant context before writing code.
2. **Scope is sacred** — build what the task says; nothing more, nothing less.
3. **Every change is verifiable** — if it can't be tested or observed, it isn't done.
4. **Clean over clever** — boring, readable code wins.
5. **One thing at a time** — never mix refactoring with feature work.
6. **Failures are expected** — handle errors as first-class flow.
7. **Fix root causes** — never patch symptoms or wrap broken code.

## TypeScript

- Strict mode (`strict: true`), `noUnusedLocals`, `noUnusedParameters`,
  `noImplicitReturns`, `noFallthroughCasesInSwitch` all on.
- No `any` — use precise types or `unknown` + narrowing. No `as` to silence errors.
- Use the path aliases (`@app/*`, `@core/*`, `@shared/*`, `@features/*`, `@env/*`) —
  not deep relative `../../..` chains.
- Explicit return types on exported functions and public service methods.
- Type every HTTP request/response against `core/models`; nothing crosses the network
  boundary as `any`.

## Components (standalone)

- **Standalone only** — every component/directive/pipe sets `standalone: true` (the
  default in modern Angular); no NgModules. Imports are declared per component.
- **OnPush change detection** on every component; drive the view with signals or the
  `async` pipe over observables — no manual change-detection workarounds.
- **Smart vs presentational** — container/page components hold state and call services;
  presentational components take `input()`s and emit `output()`s only.
- Templates use control flow (`@if`/`@for`/`@switch`) and the `async` pipe; no logic
  in templates beyond simple expressions.

## State & reactivity

- **Signals** for synchronous local/UI/view state; mutate via `set`/`update` with new
  references — never mutate the held object/array in place.
- **RxJS** for async streams (HTTP, events, debounced input). Always unsubscribe
  (`takeUntilDestroyed`, `async` pipe) — no leaked subscriptions.
- Bridge the two with `toSignal` / `toObservable`; do not maintain parallel stores.

## Services & HTTP

- **Services hold logic + HTTP.** All backend access goes through the core typed
  `api.service`; feature services wrap it. Components never inject `HttpClient`.
- **Interceptors** own cross-cutting HTTP concerns: `auth.interceptor` attaches the
  bearer token and handles refresh-retry on `401`; `error.interceptor` maps failures
  to typed errors. No hand-attached headers elsewhere.
- Map the response envelope in the service layer; never return raw `HttpResponse` or
  unwrapped envelopes to components.

## Responses & errors

- Consume the shared backend envelope: `{ success, data, meta? }` /
  `{ success: false, error: { code, message, fields? } }`. Branch on `error.code`,
  never on message text (the backend localizes messages).
- Every async UI surface handles all states: loading, empty, error, success.
- Never swallow errors; surface a user-friendly message in the UI and log context in
  dev. No empty `catch`/`error` callbacks.

## Styling & accessibility

- **Tokens only** — zero hardcoded hex, px font sizes, or magic spacing. Every visual
  value is a token from `context/ui-context.md`. This is a QA rejection, not a nit.
- Semantic HTML first; labels on inputs; `alt` on images; ARIA only where semantics
  can't express it. Visible focus, keyboard reachability, contrast ≥ 4.5:1.
- Respect `prefers-reduced-motion`; touch targets ≥ 44×44px; disable controls while
  their async action runs.

## Forms

- Angular Reactive Forms with typed form groups; validators live in the feature's
  `validators/`. Validate at the boundary; show field errors from `error.fields`.

## Testing

- **Where tests live (HARD):** every test file goes in `test/` (`main`, `app`, `setup`, `helpers`, `e2e/`), at the path that
  mirrors its source file — see `context/architecture.md` → **Test structure**. Never beside a
  source file, never inside a module/feature folder. `setup` keeps the suite
  self-contained: tests run on a clean checkout with no developer `.env`.
- Jest (or Karma/Jasmine) for unit/component specs; new behavior ships with a
  happy-path test plus at least one edge/error case. Playwright for e2e on critical
  flows (auth, primary user journeys).
- Pre-existing tests stay green; a task that breaks them is not done. Aim for the
  project's coverage bar on new code.

## Comments, TODOs, dependencies

- Comments explain *why*, where the code can't. No TODO/FIXME/HACK without a linked task.
- Prefer existing dependencies; a new one requires developer approval (note it in QA notes).
- Before using any third-party library, verify against the installed version's docs —
  APIs drift, training data goes stale.
- npm is the package manager (see `context/architecture.md`); do not introduce a
  second lockfile.
