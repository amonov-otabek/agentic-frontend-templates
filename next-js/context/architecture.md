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

Every file a Builder creates must fit this tree. New top-level folders require
developer approval first.

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
