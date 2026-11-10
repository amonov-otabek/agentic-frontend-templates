# Feature 01 — Frontend Setup & Scaffold

- Feature ID: FEAT-01
- Status: draft
- Depends on: none

> Source of truth for the **frontend foundation** of this Vite + React SPA. It stands
> up the runnable shell — stack, API client, providers, routing, theming, i18n, and the
> nginx static build — and writes **no auth or business logic**. The identity feature
> lands in FEAT-02.

## Goal

Stand up the application skeleton so the team is ready to write feature code: a strict
TypeScript Vite + React 19 SPA that boots clean, talks to the backend through one typed
envelope-aware `fetch` client, routes through a React Router data router with public /
auth / dashboard layout shells, themes light/dark from design tokens, localizes through
i18next, and builds to a static `dist/` served by `nginx:alpine` with SPA fallback.

## Scope

- **Stack & tooling** — Vite 8 (`@vitejs/plugin-react`), React 19 (StrictMode), strict
  TypeScript (`moduleResolution: bundler`, `verbatimModuleSyntax`, project refs:
  `tsconfig.app.json` / `tsconfig.node.json`), single `@/*` → `src/*` alias declared in
  both `vite.config.ts` and `tsconfig.app.json`. ESLint 10 flat config + Prettier +
  Husky + lint-staged. Package manager: pnpm (`pnpm-lock.yaml` is canonical — no second lockfile).
- **Env** — origin from `VITE_API_URL` only (public; default `http://localhost:3000`);
  no secrets in the client; `.env.local` gitignored, `.env.example` holds the public var.
- **Typed API client** (`src/shared/libs/api.ts`) — `api.get/getPaginated/post/put/patch/delete`;
  prepends `VITE_API_URL` origin, attaches the in-memory bearer token, unwraps the
  `{ success, data, meta? }` envelope, and throws a typed `ApiError(status, code, message, fields?)`
  on a non-2xx / `false` envelope / parse failure. `204` → `data: undefined`.
  Requests send `credentials: "include"`. Pagination via `?page&limit` → `meta`.
- **JWT + refresh plumbing** — in-memory access token (`setAccessToken` / `getAccessToken`);
  the auth service (`features/identity/services/auth.service.ts`) persists access + refresh
  in `localStorage` and rehydrates via `loadSession()` on startup. Refresh-token rotation
  and forced sign-out on `401/403` are wired here as the contract the identity feature uses.
- **i18n** (`src/shared/i18n`) — i18next + react-i18next + browser language detector;
  `messages/{en,ru,uz}.json`; `routing.ts` locale list; all user-facing strings resolve
  through `t(...)` — no hardcoded copy.
- **Theme / tokens + dark mode** — Tailwind CSS v4 CSS-first `@theme` in
  `src/styles/globals.css`; OKLCH token palette with `:root` (light) + `.dark` overrides;
  `next-themes` class strategy, system default; Geist Variable font. shadcn (`base-nova`)
  primitives over `@base-ui/react` under `shared/components/ui/*` (generated, not hand-edited).
- **Router** (`src/app/router.tsx`) — `createBrowserRouter` data router mounting three
  layout groups: `PublicLayout` (`/`), `AuthLayout` (`/sign-in`, `/sign-up`,
  `/forgot-password`, `/reset-password`, `/two-factor`, `/verify-email`), `DashboardLayout`
  (`/dashboard`, `/messages`, `/users`, `/notifications`, `/audit`, `/analytics`,
  `/payments`, `/settings`). Each dashboard route carries a `handle.titleKey`.
- **Layout shell** (`src/layout/*`) — public header/footer, auth layout, dashboard layout
  with app shell + sidebar + header + user nav, and a `coming-soon` placeholder.
- **Route guard** — dashboard routes check `isAuthenticated()` and redirect unauthenticated
  users to `/sign-in`; authenticated users skip the auth group. Guards are UX-only — the
  backend remains the authz source of truth (a `401/403` clears the session).
- **Providers** (`src/shared/components/provider/Providers.tsx`) — compose once in order
  i18n → theme → TanStack Query → toaster (`sonner`); mounted in `main.tsx` wrapping
  `<RouterProvider/>`.
- **Production build** — `pnpm build` (`tsc -b` project refs, then `vite build` → static
  `dist/`); multi-stage `Dockerfile` builds then serves via `nginx:alpine`; `nginx.conf`
  SPA fallback (`try_files … /index.html`) + asset caching; `docker/compose/docker-compose.{dev,staging,production}.yml`; `scripts/{deploy,health}.sh`.
- Out of scope: any auth/business logic, feature data wiring, tests (FEAT-08).

## Consumed API

- None directly. This feature defines the **client** of the backend envelope contract
  (`/api/v1`, `{ success, data, meta? }`) consumed by every later feature; it makes no
  real endpoint calls itself.

## UI / components used

- Layout shells (`src/layout/*`), `shared/components/provider/*`, `shared/components/ui/*`
  (shadcn primitives), `shared/components/widgets/{icons,locale-switch,theme-toggle,pager,sticky-header}`,
  `shared/hooks/*`, tokens in `src/styles/globals.css`.

## Acceptance criteria

- [ ] `pnpm install` succeeds; only one lockfile (`pnpm-lock.yaml`) present.
- [ ] `@/*` alias resolves identically in `vite.config.ts` and `tsconfig.app.json`.
- [ ] `shared/libs/api.ts` prepends `VITE_API_URL`, attaches the bearer token, unwraps
      `{ success, data, meta? }`, throws typed `ApiError` on failure, and resolves `204` to
      `undefined`.
- [ ] In-memory access token + `localStorage` access/refresh persistence + `loadSession()`
      hydration work; no secrets are bundled (only `VITE_API_URL`).
- [ ] i18next loads `en/ru/uz`, the language detector + locale switch work, and every
      visible string resolves through `t(...)`.
- [ ] Light/dark theme toggles via `next-themes` (system default); all visual values come
      from `globals.css` tokens (no hardcoded hex/px).
- [ ] `createBrowserRouter` mounts the public, auth, and dashboard layout groups with the
      routes listed in Scope; dashboard routes carry `handle.titleKey`.
- [ ] Dashboard routes redirect unauthenticated users to `/sign-in`; authenticated users
      are kept out of the auth group.
- [ ] Providers compose i18n → theme → query → toaster once and mount in `main.tsx`.
- [ ] `pnpm build` produces a static `dist/`; the `Dockerfile` builds it and serves via
      `nginx:alpine` with SPA fallback (`try_files … /index.html`).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
