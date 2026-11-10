# Architecture Context

HARD CONTRACT: file placement, stack choices, and boundary rules here are
non-negotiable for Builders — QA rejects violations. This is the **Next.js web client**;
keep it locked.

## Stack

| Layer                 | Technology                                              | Role               |
| --------------------- | ------------------------------------------------------- | ------------------ |
| Runtime               | Node.js ≥ 20 (standalone server, `output: 'standalone'`)| server runtime     |
| Language              | TypeScript 5 — strict, ES2017 target, `moduleResolution: bundler` | all source |
| Framework             | Next.js 16 (App Router) + React 19                      | web app            |
| Routing               | App Router under `src/app/[locale]/` with route groups `(auth)`/`(protected)`/`(public)` | pages |
| i18n                  | `next-intl` (locales `en`/`ru`/`uz`, `localePrefix: 'always'`) | localization |
| Data fetching         | TanStack Query 5 (`@tanstack/react-query`)              | server-state cache |
| HTTP client           | typed `fetch` wrapper (`shared/libs/api.ts`) over the response envelope | API access |
| Client state          | Zustand 5 (+ auto-selector helper `shared/libs/store.ts`) | local/global UI state |
| Validation            | Zod 4 (`shared/libs/validators.ts` + feature `validators/`) | form/input validation |
| Styling               | Tailwind CSS v4 (`@theme inline` in `styles/globals.css`), tokens as CSS vars (oklch) | styling |
| UI primitives         | shadcn-style components on `@base-ui/react` (`shared/components/ui/`) | component library |
| Theming               | `next-themes` (light/dark via `.dark` class)            | color mode |
| Icons / charts        | `lucide-react`, `recharts`                              | iconography, charts |
| Utilities             | `clsx` + `tailwind-merge` (`cn`), `date-fns`, `class-variance-authority` | helpers |
| Toasts                | `sonner`                                                | notifications |
| Quality               | ESLint 9 (`eslint-config-next`) + Prettier, Husky + lint-staged | lint/format/hooks |
| Build                 | `next build` → standalone server                        | compile / package |

Package manager: **pnpm** (single `pnpm-lock.yaml`; CI uses pnpm; do not introduce a second lockfile).

## Commands

| Action | Command |
| --- | --- |
| Dev server | `pnpm dev` |
| Lint | `pnpm lint` |
| Format | `pnpm format` |
| Format check | `pnpm format:check` |
| Build (native gate) | `pnpm build` (`next build`) |
| Start (prod) | `pnpm start` (`next start`) |

QA and Builders run exactly these — keep them accurate. There is no separate
typecheck/test script wired yet; `next build` is the compile gate.

## Folder structure

TypeScript path alias (tsconfig): `@/*` → `src/*`. Use it — no deep `../../..` chains
outside a feature's own internal imports.

```
src/
├── app/                          ← App Router (thin route entry points only)
│   ├── layout.tsx                ← root html shell
│   └── [locale]/
│       ├── layout.tsx            ← locale provider wrapper
│       ├── (auth)/               ← sign-in/up, forgot/reset, 2FA, verify-email + layout
│       ├── (protected)/          ← dashboard, users, messages, payments, audit, … + guard layout
│       └── (public)/             ← marketing/home + layout
├── features/<name>/              ← one slice per domain (see Feature shape)
├── layout/                       ← app shell: sidebar, headers, footers, dashboard chrome
├── shared/
│   ├── components/
│   │   ├── ui/                   ← shadcn primitives (button, card, table, dialog, …)
│   │   ├── widgets/              ← composed shared widgets (icons, locale-switch, theme-toggle, pager)
│   │   └── provider/             ← providers (query, theme, i18n) composed in providers.tsx
│   ├── hooks/                    ← use-permission, use-pagination, use-mobile, use-toast, …
│   ├── libs/                     ← api.ts (HTTP client), store.ts, validators.ts, utils.ts (cn)
│   └── i18n/                     ← routing, navigation, request + messages/{en,ru,uz}.json
├── styles/globals.css            ← Tailwind v4 @theme + token definitions (single source)
└── proxy.ts                      ← middleware (locale routing)

test/                             ← mirrors src/ (see Test structure)
Dockerfile                        ← standalone Node image
docker/compose/                   ← docker-compose.{dev,staging,production}.yml
```

### Feature shape (every `features/<name>/`)

```
api/          ← typed calls to /api/v1 via shared/libs/api (no React, no fetch by hand)
services/     ← client-side logic over api/ (token persistence, multi-step flows)
queries/      ← TanStack Query hooks (useX query/mutation) + query-key factories
store/        ← Zustand stores for this slice's UI/session state
validators/   ← Zod schemas, one per input surface
types/        ← TS types/DTOs for this slice (mirror backend shapes)
components/   ← presentational + form components for this slice
pages/        ← page-level compositions imported by app/ route files
utils/        ← slice-local helpers (optional)
```

Every file a Builder creates must fit this tree; test files follow
**Test structure** below. New top-level folders require
developer approval first.

## Test structure (HARD)

Applies to every project built from this template. QA rejects a task whose test files break any rule below.

```
src/features/auth/validators/sign-in.ts           → test/features/auth/validators/sign-in.spec.ts
src/features/auth/components/sign-in-form.tsx     → test/features/auth/components/sign-in-form.spec.tsx
src/shared/hooks/use-permission.ts                → test/shared/hooks/use-permission.spec.ts
```

```
test/
├── helpers.tsx                ← render-with-providers, query client, mocks, fixtures
├── features/<name>/           ← mirrors the slice: api/, services/, queries/, store/, validators/, components/
├── layout/                    ← mirrors src/layout
├── shared/                    ← mirrors src/shared: components/, hooks/, libs/, i18n/
└── e2e/                       ← <flow>.e2e-spec.ts (Playwright)
```

1. **One location only.** Every test file lives under `test/` — never next to a source file, never inside a module/feature folder, never in a second test folder.
2. **Mirror the source tree.** `test/` mirrors `src/` one-to-one. Same folder names, same file names, plus the `.spec.ts` / `.spec.tsx` suffix.
3. **One spec per source file**, at its mirrored path. Moving or renaming a source file moves or renames its spec in the same task.
4. **No catch-all spec.** Logic shared across the app is tested at the mirrored path of the file that owns it — never in a "global" / "misc" / "utils" spec file.
5. **Shared test code** lives in `test/helpers.tsx` (render-with-providers, query client, mocks, fixtures). Specs never import from other specs.
6. **End-to-end tests** live only in `test/e2e/` (Playwright), named `<flow>.e2e-spec.ts`. Unit/integration specs never go there, and e2e specs never go anywhere else.
7. **Runner config follows the folder.** No test runner is wired yet — when one is added (Vitest + Testing Library, Playwright), its include glob is `test/**/*.spec.{ts,tsx}` (excluding `test/e2e/`), Playwright `testDir` is `test/e2e`, coverage measures `src/`, and the commands go into the table above.
8. **Legacy layout is drift, not precedent.** If existing code or a feature spec places tests elsewhere (e.g. a module's own `tests/` folder), new and modified tests still go to the mirrored path; moving the old ones is its own task — surface the drift to the developer, never silently mass-move files.

## Boundary rules (HARD)

1. **App routes are thin** — `app/**/page.tsx` re-exports or renders a feature `pages/`
   component; no section JSX or data logic inline in route files.
2. **Pages orchestrate, components render** — a feature `pages/` composes `components/`;
   it never inlines section markup. Extract concerns to `components/<Concern>.tsx`.
3. **No business logic in components** — components render props/state; data fetching and
   logic live in `queries/`, `services/`, and `store/`.
4. **All HTTP goes through the typed client** (`shared/libs/api.ts`) inside a feature's
   `api/` folder. No bare `fetch` in components or pages.
5. **Server vs client components** — default to Server Components; add `"use client"` only
   where hooks/interactivity/Query are needed (forms, query hooks, stores).
6. **Tokens only for styling** — every visual value resolves to a token from
   `context/ui-context.md` (CSS vars in `globals.css`). No hardcoded hex/px/spacing.
7. **Never mutate** — return new objects for state updates (immutability everywhere).
8. **Cross-feature reuse goes through `shared/`** — features do not import each other's
   internals; lift shared concerns into `shared/components` / `shared/libs`.

## Cross-cutting invariants

- **Response envelope** is the only shape the client trusts: success `{ success, data, meta? }`,
  error `{ success, error: { code, message, fields? } }`. The client unwraps `data` and throws
  a typed `ApiError(status, code, message, fields)` on failure.
- **Auth**: JWT bearer; access token held in memory + persisted to `localStorage`, set on the
  client via `setAccessToken`. Refresh token persisted for re-auth. 2FA challenge stashed in
  `sessionStorage` across the sign-in → two-factor hop.
- **Route guards**: the `(protected)` layout redirects unauthenticated users to `(auth)`.
- **i18n**: every user-facing string resolves through `next-intl`; URLs are locale-prefixed.
- **Query keys**: each feature owns a key factory; mutations invalidate by key — no manual cache writes except deliberate `setQueryData` priming.

## Decisions log

Locked decisions agents must not re-litigate.

- **App Router + feature slices** — routing in `app/`, all real code in `features/<name>/`
  by layer; route files stay thin.
- **TanStack Query for server state, Zustand for client state** — do not store server data
  in Zustand or hand-roll fetching in components.
- **Typed envelope client** — one `fetch` wrapper unwraps the backend envelope and normalizes
  errors to `ApiError`; nothing bypasses it.
- **Tailwind v4 tokens via `@theme`** — colors are oklch CSS vars with light/dark sets; the
  design system in `context/ui-context.md` is the single source of visual truth.
- **shadcn on `@base-ui/react`** — UI primitives live in `shared/components/ui`; do not add a
  second component library.
