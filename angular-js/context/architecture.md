# Architecture Context

HARD CONTRACT: file placement, stack choices, and boundary rules here are
non-negotiable for Builders — QA rejects violations. This is the planned foundation
for an empty Angular SPA project; keep it locked once the first feature is built.

## Stack

| Layer                 | Technology                                              | Role               |
| --------------------- | ------------------------------------------------------- | ------------------ |
| Framework             | Angular (latest) — **standalone components, no NgModules** | SPA framework      |
| Language              | TypeScript 5.x — `strict: true`, no `any`              | all source         |
| Routing               | Angular Router (lazy, standalone route configs)         | navigation, guards |
| State                 | Angular **signals** for local/UI + view state; **RxJS** for async streams | reactivity |
| HTTP                  | `HttpClient` behind a typed API service + interceptors  | backend access     |
| Auth                  | JWT bearer via HTTP interceptor (attach + refresh-retry)| identity           |
| Styling               | SCSS with design tokens as **CSS custom properties** (Tailwind optional, tokens-first) | styling |
| Forms                 | Angular Reactive Forms + typed validators               | input handling     |
| Build                 | Angular CLI (`ng build`) → static `dist/`               | compile/bundle     |
| Unit/component tests  | Jest (or Karma/Jasmine) via `ng test`                   | tests              |
| E2E tests             | Playwright                                              | end-to-end         |
| Quality               | ESLint (`angular-eslint`) + Prettier, Husky + lint-staged | lint/format/hooks |
| Static host           | `nginx:alpine` serving `dist/` with SPA fallback        | production         |

Package manager: **npm** is canonical (matches the SPA siblings and Docker build).
Pick one lockfile for the project and do not mix.

## Commands

| Action | Command |
| --- | --- |
| Dev server | `ng serve` (HMR on `:4200`) |
| Lint | `ng lint` |
| Format check | `npm run format:check` |
| Build (native gate) | `ng build` (production static `dist/`) |
| Unit/component tests | `ng test` (Jest/Karma) |
| E2E | `npm run e2e` (Playwright) |
| Production image (build) | `docker build -t angular-web .` |
| Full dev stack | `docker compose -f docker/compose/docker-compose.dev.yml up` |

QA and Builders run exactly these — keep them accurate.

## Folder structure

TypeScript path aliases (tsconfig `paths`): `@app/*`, `@core/*`, `@shared/*`,
`@features/*`, `@env/*`.

```
src/
├── main.ts                     ← bootstrapApplication(AppComponent, appConfig)
├── app/
│   ├── app.component.ts         ← root shell (router-outlet + layout)
│   ├── app.config.ts            ← providers: router, HttpClient, interceptors
│   ├── app.routes.ts            ← top-level routes (lazy feature routes)
│   ├── core/                    ← app-wide singletons (provided in root)
│   │   ├── http/                ← api.service (typed), http error mapping
│   │   ├── interceptors/        ← auth.interceptor, error.interceptor
│   │   ├── guards/              ← auth.guard, role.guard (CanActivateFn)
│   │   ├── auth/                ← auth.service, token store (signals)
│   │   └── models/              ← shared API/domain TS interfaces + envelope types
│   ├── shared/                  ← presentational, reusable, stateless
│   │   ├── components/          ← buttons, cards, inputs, badges (design system)
│   │   ├── directives/ pipes/   ← cross-cutting view helpers
│   │   └── tokens/              ← design-token SCSS + a11y utilities
│   └── features/<name>/         ← one folder per feature (see Feature shape)
├── styles/                      ← global styles, token definitions (CSS custom props)
└── environments/                ← environment.ts / environment.prod.ts (API base URL)

Dockerfile                       ← multi-stage: build (node) → nginx:alpine static serve
nginx.conf                       ← SPA fallback (try_files → /index.html)
docker/
└── compose/                     ← docker-compose.{dev,staging,production}.yml
```

### Feature shape (every `features/<name>/`)

```
<name>.routes.ts   ← lazy route config for the feature
pages/             ← smart/container components (route targets; hold state, call services)
components/         ← presentational components for this feature
services/          ← feature logic + HTTP calls (via the core api.service)
models/            ← feature TS interfaces
validators/        ← reactive-form validators / Zod-equivalent schemas
tests/             ← component + service specs
```

Every file a Builder creates must fit this tree. New top-level folders require
developer approval first.

## Boundary rules (HARD)

1. **Smart vs presentational.** Page/container components own state and call services;
   presentational components in `shared/`/`components/` take inputs and emit outputs
   only — no HTTP, no service injection beyond pure view helpers.
2. **Services hold logic + HTTP.** All backend access goes through the core typed
   `api.service` (and feature services that wrap it). Components never call `HttpClient`
   directly.
3. **Typed API only.** Every request/response is typed against `core/models`; never
   pass `any` across the HTTP boundary. Map the envelope, never leak raw responses.
4. **Tokens-only styling.** No hardcoded colors/sizes/spacing — every visual value is
   a token (see `context/ui-context.md`).
5. **Auth via interceptor.** Token attach + refresh-retry live in `auth.interceptor`;
   components and services never hand-attach the bearer header.
6. **Immutability everywhere.** Never mutate state objects/arrays — produce new values
   (signals `set`/`update` with new references; RxJS operators, not in-place mutation).
7. **Lazy by default.** Feature routes are lazy-loaded; the root bundle stays lean.

## Cross-cutting invariants

- **Change detection**: components use `ChangeDetectionStrategy.OnPush`; state is
  signals or observables (with `async` pipe) — no manual `markForCheck` hacks.
- **Environment config**: API base URL and flags come from `environments/*` only;
  no hardcoded URLs in components/services.
- **Response envelope** (mirrors the backend, defined in `core/models`):
  - success: `{ success: true, data, meta? }`
  - error: `{ success: false, error: { code, message, fields? } }`
  - paginated: `data` is the items array, `meta: { page, limit, total, totalPages }`
- **Error handling**: the HTTP error interceptor maps failures to typed errors; UI
  surfaces user-friendly messages and never swallows errors silently.
- **Accessibility**: semantic HTML, visible focus, keyboard reachability, and
  contrast are part of "done" — QA checks them.

## Decisions log

Locked decisions agents must not re-litigate.

- **Standalone components, no NgModules** — every component/directive/pipe is
  standalone; bootstrap via `bootstrapApplication` + `app.config.ts` providers.
- **Signals + RxJS split** — signals for synchronous view/local state; RxJS for async
  streams (HTTP, events). Bridge with `toSignal`/`toObservable`, not parallel stores.
- **Typed API service + interceptors** — one place for HTTP shape and auth; components
  stay framework-thin.
- **Tokens-first styling** — design tokens as CSS custom properties are the single
  source of visual truth; Tailwind, if used, is wired to the same tokens.
- **Static SPA on nginx** — built with `ng build`, served by `nginx:alpine` with SPA
  fallback (consistent with the Vite SPA siblings).
