# Architecture Context

HARD CONTRACT: file placement, stack choices, and boundary rules here are
non-negotiable for Builders — QA rejects violations. Derived from the `vue-js`
reference template; keep it locked.

## Stack

| Layer                 | Technology                                              | Role               |
| --------------------- | ------------------------------------------------------- | ------------------ |
| Build tool            | Vite 8 (`@vitejs/plugin-vue`)                           | dev server + bundler |
| Framework             | Vue 3.5 — Composition API, `<script setup>`             | UI                 |
| Language              | TypeScript ~6.0 — strict via `@vue/tsconfig`            | all source         |
| Routing               | Vue Router 5 (`createWebHistory`)                       | client-side routing |
| Server state          | TanStack Query (`@tanstack/vue-query`)                  | fetching, caching, mutations |
| Client state          | Pinia 3 (setup stores)                                  | cross-cutting client state |
| HTTP client           | typed `fetch` wrapper (`shared/libs/api.ts`)            | API access over the envelope |
| Styling               | Tailwind CSS v4 (`@tailwindcss/vite`) + CSS tokens      | styling            |
| UI library            | shadcn-vue + reka-ui (`shared/components/ui/*`)         | primitives         |
| Forms / validation    | vee-validate + `@vee-validate/zod` + Zod               | form state + schema validation |
| i18n                  | vue-i18n 11 (Composition mode, en/ru/uz)                | localization       |
| Icons                 | lucide (`@lucide/vue`)                                   | iconography        |
| Tables / charts       | `@tanstack/vue-table`, `@unovis/vue`                     | data grids, charts |
| Toasts                | vue-sonner                                              | notifications      |
| Utilities             | `@vueuse/core`, clsx, tailwind-merge (`cn`)             | composables, class merge |
| Lint / format         | ESLint 10 + `eslint-plugin-vue` + Prettier; Husky + lint-staged | quality |
| Runtime (prod)        | static build served by `nginx:1.27-alpine` (SPA fallback) | hosting          |

Package manager: **pnpm** (`pnpm-lock.yaml`; Dockerfile uses `corepack` + `pnpm install --frozen-lockfile`). One lockfile only — do not introduce npm/yarn locks.

## Commands

| Action | Command |
| --- | --- |
| Dev server | `pnpm dev` (Vite) |
| Lint | `pnpm lint` (`eslint .`) |
| Format | `pnpm format` / `pnpm format:check` |
| Typecheck + Build (native gate) | `pnpm build` (`vue-tsc -b && vite build` → `dist/`) |
| Preview prod build | `pnpm preview` |
| Docker build (static → nginx) | `docker build -t vue-js .` |
| Full stack (compose) | `docker compose -f docker/compose/docker-compose.dev.yml up` |

QA and Builders run exactly these — keep them accurate. `pnpm build` is the typecheck
gate (`vue-tsc`); there is no separate typecheck script.

## Folder structure

Path alias (`vite.config.ts` + tsconfig): `@/*` → `src/*`. No deep `../../..` chains.

```
src/
├── main.ts                    ← bootstrap: createApp + providers + router, mount #app
├── middleware.ts              ← inert placeholder (no server layer in a Vite SPA)
├── app/                       ← App.vue, router.ts, HomeView.vue, DashboardView.vue
├── layout/                    ← public/auth/dashboard shells, headers, sidebar, footer
├── features/<name>/           ← one folder per domain feature (see Feature shape)
├── shared/
│   ├── components/
│   │   ├── ui/                ← shadcn-vue primitives (one folder per component)
│   │   ├── widgets/           ← app-level widgets (icons, pager, theme-toggle, …)
│   │   └── provider/          ← providers.ts, query-provider, i18n-provider, theme-provider.vue
│   ├── hooks/                 ← composables (use-toast, use-pagination, use-permission, …)
│   ├── libs/                  ← api.ts (HTTP client), store.ts (pinia), utils.ts (cn), validators.ts
│   └── i18n/                  ← createI18n + messages/{en,ru,uz}.json
└── styles/                    ← globals.css (Tailwind v4 + design tokens)

test/                          ← mirrors src/ (see Test structure)
index.html · nginx.conf · Dockerfile · components.json (shadcn-vue)
docker/compose/docker-compose.{dev,staging,production}.yml
```

### Feature shape (every `features/<name>/`)

```
api/         ← thin functions wrapping the api client; backend paths live here only
queries/     ← TanStack Query hooks (useQuery/useMutation) + query keys
store/        ← Pinia setup stores (only when client state is needed; optional)
services/    ← non-React orchestration (e.g. token persistence; optional)
types/       ← TS types for DTOs and view models
validators/  ← Zod schemas for forms (optional)
components/   ← feature .vue components (render only)
pages/       ← route-level .vue pages (orchestrate components + queries)
```

Every file a Builder creates must fit this tree; test files follow
**Test structure** below. New top-level folders require
developer approval first.

## Test structure (HARD)

Every project built from this template uses this exact test layout. QA rejects a task whose test files break any rule below.

```
test/
├── main.spec.ts               ← bootstrap: createApp + plugins
├── app.spec.ts                ← app shell: App.vue, router, layouts
├── setup.ts                   ← self-contained test env (runner setup file)
├── helpers.ts                 ← render-with-providers, fixtures, mocks
├── features/<name>/           ← mirrors the feature: api/, queries/, store/, components/, …
├── layout/                    ← mirrors src/layout
├── shared/                    ← mirrors src/shared: components/, hooks/, libs/, i18n/
└── e2e/                       ← <flow>.e2e-spec.ts
```

Source → spec mapping:

```
src/main.ts                                       → test/main.spec.ts
src/features/auth/components/SignInForm.vue       → test/features/auth/components/SignInForm.spec.ts
src/shared/hooks/use-pagination.ts                → test/shared/hooks/use-pagination.spec.ts
```

1. **One location only.** Every test file lives under `test/` at the project root — never next to a source file, never inside a module/feature folder, never in a second test folder.
2. **Fixed root files — nothing else at the root.** `main.spec.ts` covers bootstrap only: `createApp` with every plugin installs and mounts. `app.spec.ts` covers the app shell only: `App.vue`, `router.ts` layouts, guards, not-found. `setup.ts` makes the suite self-contained: DOM matchers, global mocks (fetch/API client, storage, `matchMedia`, i18n), test env values — loaded by the runner as its setup file; no developer `.env` is required. `helpers.ts` holds mount-with-plugins (pinia, router, i18n, query client), fixtures and mocks — no assertions. Besides these four, the root holds only mirrored folders and `e2e/`.
3. **Mirror the source tree.** Every other spec sits at the path of the file it tests: same folder names, same file name, plus the `.spec.ts` suffix (`*.test.ts` is not used). (`SignInForm.vue` → `SignInForm.spec.ts`)
4. **One spec per source file.** Moving or renaming a source file moves or renames its spec in the same task.
5. **No catch-all spec.** Logic shared across the app is tested at the mirrored path of the file that owns it — never in a "global" / "misc" / "utils" spec.
6. **Specs import only source code and the helpers file** — never another spec, never a fixture defined inside another spec.
7. **End-to-end tests** live only in `test/e2e/` (Playwright), named `<flow>.e2e-spec.ts`. Unit/integration specs never go there; the e2e folder may hold one `helpers` file of its own (route/bridge mocks, launch fixtures).
8. **Runner config follows the folder.** No test runner is wired yet. When one is added (Vitest + Vue Test Utils, Playwright): include `test/**/*.spec.ts`, exclude `test/e2e/**`, `setupFiles: ['test/setup.ts']`, Playwright `testDir: ./test/e2e`, coverage measures `src/`; then add the commands to the table above. If the config disagrees with this section, fix the config in the same task.
9. **Same tree in every template.** The root file names (`main`, `app`, `setup`, `helpers`, `e2e/`) are identical across all stacks, so a developer moving between a TS and a Python API finds the same layout. A change to this rule is applied to every template at once.
10. **Legacy layout is drift, not precedent.** If existing code or a feature spec places tests elsewhere (e.g. a module's own `tests/` folder or `*.test.ts` files), new and modified tests still follow this section; moving the old ones is its own task — surface the drift to the developer, never silently mass-move files.

## Boundary rules (HARD)

1. **Pages orchestrate, components render.** Pages compose feature components and wire
   queries; components receive props/emit events and render. No page-section JSX inline.
2. **All HTTP goes through `shared/libs/api.ts`.** Feature `api/*.ts` functions are the
   only place backend paths (`/api/v1/...`) appear. Components never call `fetch`.
3. **Server state via TanStack Query only** (`queries/*.queries.ts`). No manual cache,
   no `fetch` in components. Client state via Pinia setup stores.
4. **Tokens only** — every color/spacing/radius/font value comes from a token in
   `src/styles/globals.css`. Zero hardcoded hex/px (QA rejection).
5. **Typed end-to-end** — DTOs and view models in `features/<name>/types`; the api
   client returns typed data and throws `ApiError`. No `any`.
6. Features do not import each other's stores/services directly — share via
   `shared/*` or compose at the page level.
7. **Immutability everywhere** — return new objects; never mutate props, store state
   in place (use store actions), or query data.

## Cross-cutting invariants

- **Env at build time** via Vite: only `import.meta.env.VITE_*` vars exist in the
  bundle. `VITE_API_URL` is the backend origin (default `http://localhost:3000`); the
  `/api/v1` prefix is added by feature api functions.
- **Auth token**: stored in localStorage by `auth.service`, mirrored into the in-memory
  bearer (`setAccessToken`) attached by the api client on every request.
- **Route guard**: `router.beforeEach` redirects `requiresAuth` routes to `/sign-in`
  when no session is stored.
- **Response envelope** (`shared/libs/api.ts`):
  - success: `{ success: true, data, meta? }`
  - error: `{ success: false, error: { code, message, fields? } }` → thrown as `ApiError`
  - paginated: `api.getPaginated` returns `{ data, meta: { total, page, limit } }`
- All user-facing strings go through `vue-i18n` (`shared/i18n`).

## Decisions log

Locked decisions agents must not re-litigate.

- **Static SPA, no SSR** — Vite build served by nginx with SPA fallback; the
  `middleware.ts` placeholder exists only for cross-template folder parity.
- **TanStack Query owns server state; Pinia owns client state** — no overlap, no
  bespoke caching layer.
- **shadcn-vue is the component source** — primitives are vendored under
  `shared/components/ui`; use/extend them, do not add a second UI library.
- **Typed fetch wrapper over the envelope** — no axios; one `api` object, one
  `ApiError` type.
