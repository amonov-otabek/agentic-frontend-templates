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

test/                            ← mirrors src/ (see Test structure)
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
```

Every file a Builder creates must fit this tree; test files follow
**Test structure** below. New top-level folders require
developer approval first.

## Test structure (HARD)

Applies to every project built from this template. QA rejects a task whose test files break any rule below.

```
src/app/core/auth/auth.service.ts                 → test/app/core/auth/auth.service.spec.ts
src/app/features/users/pages/users.page.ts        → test/app/features/users/pages/users.page.spec.ts
src/app/shared/components/button/button.component.ts
  → test/app/shared/components/button/button.component.spec.ts
```

```
test/
├── helpers.ts                 ← TestBed setup, providers, mocks, fixtures
├── app/
│   ├── core/                  ← mirrors src/app/core: http/, interceptors/, guards/, auth/
│   ├── shared/                ← mirrors src/app/shared: components/, directives/, pipes/
│   └── features/<name>/       ← mirrors the feature: pages/, components/, services/, validators/
└── e2e/                       ← <flow>.e2e-spec.ts (Playwright)
```

1. **One location only.** Every test file lives under `test/` — never next to a source file, never inside a module/feature folder, never in a second test folder.
2. **Mirror the source tree.** `test/` mirrors `src/` one-to-one. Same folder names, same file names, plus the `.spec.ts` suffix. This overrides the Angular CLI default of a spec beside each component — generate with `--skip-tests` and create the spec in `test/`.
3. **One spec per source file**, at its mirrored path. Moving or renaming a source file moves or renames its spec in the same task.
4. **No catch-all spec.** Logic shared across the app is tested at the mirrored path of the file that owns it — never in a "global" / "misc" / "utils" spec file.
5. **Shared test code** lives in `test/helpers.ts` (TestBed setup, providers, mocks, fixtures). Specs never import from other specs.
6. **End-to-end tests** live only in `test/e2e/` (Playwright), named `<flow>.e2e-spec.ts`. Unit/integration specs never go there, and e2e specs never go anywhere else.
7. **Runner config follows the folder.** The unit runner includes only `test/**/*.spec.ts` (excluding `test/e2e/`); Playwright `testDir` is `test/e2e`; coverage measures `src/`. If the config disagrees, fix it in the same task.
8. **Legacy layout is drift, not precedent.** If existing code or a feature spec places tests elsewhere (e.g. a module's own `tests/` folder), new and modified tests still go to the mirrored path; moving the old ones is its own task — surface the drift to the developer, never silently mass-move files.

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
