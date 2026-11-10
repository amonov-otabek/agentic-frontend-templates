# Architecture Context

HARD CONTRACT: file placement, stack choices, and boundary rules here are
non-negotiable for Builders — QA rejects violations. This is the **Vite + React SPA**
client for `express-monolith-api`; keep it locked.

## Stack

| Layer                 | Technology                                              | Role               |
| --------------------- | ------------------------------------------------------- | ------------------ |
| Build tool / dev server | Vite 8 (`@vitejs/plugin-react`)                       | bundle + HMR       |
| Language              | TypeScript ~6.0 — strict, `moduleResolution: bundler`, `verbatimModuleSyntax`, `noEmit` | all source |
| UI library            | React 19 (`react-dom/client`, StrictMode)              | rendering          |
| Routing               | React Router 7 (`createBrowserRouter` / data router)   | client-side routes |
| Styling               | Tailwind CSS v4 (`@tailwindcss/vite`, CSS-first `@theme`) | tokens + utilities |
| Component primitives   | shadcn (`style: base-nova`) over `@base-ui/react`; `class-variance-authority`, `clsx`, `tailwind-merge` | UI kit |
| Icons                 | `lucide-react` (wrapped in `shared/components/widgets/Icons.tsx`) | iconography |
| Server state          | TanStack Query 5 (`@tanstack/react-query`)             | data fetching/cache |
| Client state          | Zustand 5 (`shared/libs/store.ts` + `createSelectors`) | local/UI/auth state |
| HTTP client           | native `fetch` wrapper (`shared/libs/api.ts`)          | typed API calls    |
| Validation            | Zod 4                                                   | form + input validation |
| Forms                 | controlled React + Zod validators (`features/*/validators`) | form handling |
| i18n                  | `i18next` + `react-i18next` + browser language detector | localization (en/ru/uz) |
| Theming               | `next-themes` (class strategy, system default)         | light / dark mode  |
| Fonts                 | `@fontsource-variable/geist` (Geist Variable)          | typography         |
| Toasts                | `sonner`                                                | notifications      |
| Quality               | ESLint 10 (flat config) + Prettier + Husky + lint-staged | lint/format/hooks |
| Serve (prod)          | `nginx:alpine` static server (`nginx.conf`, SPA fallback) | production hosting |

Package manager: a `pnpm-lock.yaml` is present and is canonical for this template.
Do not introduce a second lockfile.

## Commands

| Action | Command |
| --- | --- |
| Dev server | `pnpm dev` (Vite, HMR) |
| Lint | `pnpm lint` (ESLint) |
| Format | `pnpm format` / check: `pnpm format:check` |
| Typecheck + Build (native gate) | `pnpm build` (`tsc -b` project refs, then `vite build` → static `dist/`) |
| Preview prod build | `pnpm preview` |

The **build is the typecheck gate**: `tsc -b` runs across `tsconfig.app.json` /
`tsconfig.node.json` before Vite emits. The static `dist/` is what nginx serves in
production (Dockerfile builds it, then copies into `nginx:alpine`). QA and Builders run
exactly these — keep them accurate.

## Folder structure

TypeScript / Vite path alias: `@/*` → `src/*` (one alias only — defined in both
`vite.config.ts` and `tsconfig.app.json`).

```
src/
├── main.tsx                  ← bootstrap: createRoot → <Providers><RouterProvider/></Providers>
├── middleware.ts             ← inert placeholder (SPAs have no server middleware; see file)
├── app/
│   ├── router.tsx            ← createBrowserRouter; mounts every layout + page
│   ├── HomePage.tsx          ← public landing
│   └── DashboardPage.tsx     ← dashboard index
├── layout/                   ← shell chrome: public/auth/dashboard layouts, headers, sidebar, footer
├── features/<name>/          ← one folder per backend domain module (see Feature shape)
├── shared/
│   ├── components/
│   │   ├── ui/               ← shadcn primitives (button, card, table, dialog, …) — do not hand-edit
│   │   ├── widgets/          ← app-specific shared widgets (icons, pager, theme-toggle, …)
│   │   └── provider/         ← Providers, query/theme/i18n providers
│   ├── hooks/                ← cross-feature hooks (use-mobile, usePagination, usePermission, …)
│   ├── i18n/                 ← i18next setup + messages/{en,ru,uz}.json + routing
│   └── libs/                 ← api.ts (HTTP client), store.ts (zustand), utils.ts (cn), validators.ts
└── styles/
    └── globals.css           ← Tailwind v4 entry + @theme tokens + :root / .dark palettes

test/                         ← mirrors src/ (see Test structure)
Dockerfile                    ← multi-stage: build (pnpm build) → nginx:alpine serve
nginx.conf                    ← SPA fallback (try_files … /index.html) + asset caching
docker/compose/               ← docker-compose.{dev,staging,production}.yml (compose-only)
scripts/                      ← deploy.sh, health.sh
```

### Feature shape (every `features/<name>/`)

```
api/          ← thin functions over shared/libs/api (one file per backend resource)
queries/      ← TanStack Query hooks + a typed query-key factory (useX / useCreateX)
types/        ← TS interfaces mirroring backend models + DTOs
components/    ← presentational feature components (render-only)
pages/        ← route entry components (orchestrate: hooks → compose components)
validators/   ← Zod schemas for this feature's forms (optional)
services/     ← non-React orchestration (e.g. token persistence) (optional)
store/        ← zustand slice for feature-local state (optional)
```

Every file a Builder creates must fit this tree; test files follow
**Test structure** below. New top-level folders require
developer approval first.

## Test structure (HARD)

Every project built from this template uses this exact test layout. QA rejects a task whose test files break any rule below.

```
test/
├── main.spec.tsx              ← bootstrap: createRoot + Providers
├── app.spec.tsx               ← app shell: router, layouts, guards
├── setup.ts                   ← self-contained test env (runner setup file)
├── helpers.tsx                ← render-with-providers, fixtures, mocks
├── features/<name>/           ← mirrors the feature: api/, queries/, components/, pages/, …
├── layout/                    ← mirrors src/layout
├── shared/                    ← mirrors src/shared: components/, hooks/, libs/, i18n/
└── e2e/                       ← <flow>.e2e-spec.ts
```

Source → spec mapping:

```
src/main.tsx                                      → test/main.spec.tsx
src/features/auth/queries/useSignIn.ts            → test/features/auth/queries/useSignIn.spec.ts
src/shared/libs/api.ts                            → test/shared/libs/api.spec.ts
```

1. **One location only.** Every test file lives under `test/` at the project root — never next to a source file, never inside a module/feature folder, never in a second test folder.
2. **Fixed root files — nothing else at the root.** `main.spec.tsx` covers bootstrap only: the root mounts with the full provider stack. `app.spec.tsx` covers the app shell only: `app/router.tsx` layouts, guards, not-found. `setup.ts` makes the suite self-contained: DOM matchers, global mocks (fetch/API client, storage, `matchMedia`, i18n), test env values — loaded by the runner as its setup file; no developer `.env` is required. `helpers.tsx` holds render-with-providers (router, query client, i18n, theme), fixtures and mocks — no assertions. Besides these four, the root holds only mirrored folders and `e2e/`.
3. **Mirror the source tree.** Every other spec sits at the path of the file it tests: same folder names, same file name, plus the `.spec.ts` / `.spec.tsx` suffix (`*.test.*` is not used).
4. **One spec per source file.** Moving or renaming a source file moves or renames its spec in the same task.
5. **No catch-all spec.** Logic shared across the app is tested at the mirrored path of the file that owns it — never in a "global" / "misc" / "utils" spec.
6. **Specs import only source code and the helpers file** — never another spec, never a fixture defined inside another spec.
7. **End-to-end tests** live only in `test/e2e/` (Playwright), named `<flow>.e2e-spec.ts`. Unit/integration specs never go there; the e2e folder may hold one `helpers` file of its own (route/bridge mocks, launch fixtures).
8. **Runner config follows the folder.** No test runner is wired yet. When one is added (Vitest + Testing Library, Playwright): include `test/**/*.spec.{ts,tsx}`, exclude `test/e2e/**`, `setupFiles: ['test/setup.ts']`, Playwright `testDir: ./test/e2e`, coverage measures `src/`; then add the commands to the table above. If the config disagrees with this section, fix the config in the same task.
9. **Same tree in every template.** The root file names (`main`, `app`, `setup`, `helpers`, `e2e/`) are identical across all stacks, so a developer moving between a TS and a Python API finds the same layout. A change to this rule is applied to every template at once.
10. **Legacy layout is drift, not precedent.** If existing code or a feature spec places tests elsewhere (e.g. a module's own `tests/` folder or `*.test.ts` files), new and modified tests still follow this section; moving the old ones is its own task — surface the drift to the developer, never silently mass-move files.

## Boundary rules (HARD)

1. **Pages orchestrate, components render.** A page composes feature components and
   wires data hooks; it holds no inline section JSX and no API/`fetch` calls.
2. **Components are presentational** — they take typed props and render. No data
   fetching, no business logic, no direct API calls inside components.
3. **All HTTP goes through `shared/libs/api.ts`.** Feature `api/` modules call `api.*`;
   nothing else calls `fetch`. Queries/mutations live in `queries/`, never in JSX.
4. **Tokens only.** Every color, font size, radius, and spacing value comes from a
   token (`styles/globals.css` / `context/ui-context.md`). No hardcoded hex/px.
5. **Typed end to end.** API responses are typed via feature `types/`; no `any`, no
   raw `unknown` leaking into components.
6. **Never mutate state** — return new objects/arrays (immutability everywhere; this
   is what lets React + Query diff correctly).
7. Cross-feature reuse goes through `shared/` — features do not import each other's
   internals.

## Cross-cutting invariants

- **Single API client**: `shared/libs/api.ts` adds `VITE_API_URL` origin + `/api/v1`
  paths, attaches the in-memory bearer token, unwraps the envelope, and throws a typed
  `ApiError(status, code, message, fields?)` on failure.
- **Auth token**: held in memory (`setAccessToken`) + persisted in `localStorage`
  (access/refresh) by `features/identity/services/auth.service.ts`; hydrated on startup
  via `loadSession()`.
- **Route guards** are UX-only: dashboard routes check `isAuthenticated()` and redirect
  to `/sign-in`. The backend is the source of truth for authz.
- **Providers** are composed once in `shared/components/provider/Providers.tsx`
  (i18n → theme → query → toaster) and mounted in `main.tsx`.
- **Response envelope** (mirrors backend, `shared/libs/api.ts`):
  - success: `{ success: true, data, meta? }`
  - error: `{ success: false, error: { code, message, fields? } }`
  - paginated: `data` is the items array, `meta: { total, page, limit }`
- All user-facing strings resolve through `react-i18next` (`shared/i18n`).

## Decisions log

Locked decisions agents must not re-litigate.

- **Vite static SPA over SSR** — built to `dist/`, served by `nginx:alpine` with SPA
  fallback. SSR is the `next-js` template's job, not this one.
- **TanStack Query for server state, Zustand for client state** — never store server
  data in Zustand or refetch by hand; query keys + invalidation own the cache.
- **Single typed `fetch` client** — no Axios; one envelope-aware wrapper, one alias.
- **shadcn (base-nova) + Tailwind v4 tokens** — primitives live in `shared/components/ui`
  and are generated, not hand-authored; visual values come from `@theme`.
