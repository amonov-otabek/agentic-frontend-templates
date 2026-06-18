# Feature 01 — Frontend Setup & Scaffold

- Feature ID: FEAT-01
- Status: draft
- Depends on: none

> Source of truth for the **frontend foundation** of this Vite + Vue 3 SPA
> template. `/parse-features 01` turns the `## Tasks` section below into
> `context/tasks-list/01-setup-frontend-tasks.md`. This feature stands up a
> runnable, production-grade shell — providers, routing, theming, i18n, and the
> typed API client — but writes **no auth or domain logic**. Identity flows land
> in FEAT-02; each consumed domain follows in FEAT-03…07.

## Goal

Stand up the SPA foundation so the team is ready to write feature code: a
strict-TypeScript Vue 3 app bootstrapped by Vite, wired with Pinia + TanStack
Query providers, vue-router with auth/public/dashboard layout shells and a guard,
a typed envelope API client (in-memory bearer + localStorage refresh), vue-i18n
(en/ru/uz), token-based theming with light/dark, and a static `nginx` build with
SPA fallback.

## Scope

- **Bootstrap** (`src/main.ts`): `createApp(App).use(providers).use(router)`, mount
  `#app`, import `@/styles/globals.css`.
- **Providers** (`src/shared/components/provider/`): `providers.ts` registers Pinia
  (`@/shared/libs/store`), vue-i18n (`@/shared/i18n`), and `VueQueryPlugin` with a
  shared `queryClient` (`query-provider.ts`); `theme-provider.vue` toggles
  `.light`/`.dark` on `<html>` and persists to `localStorage('ui-theme')`,
  defaulting to `system`.
- **Typed API client** (`src/shared/libs/api.ts`): `ApiEnvelope<T>` =
  `{ success:true, data, meta? }` | `{ success:false, error:{ code, message, fields? } }`;
  `api.get/getPaginated/post/put/patch/delete`; `ApiError` (status, code, fields);
  base URL from `VITE_API_URL` (origin only, `/api/v1` added per call);
  in-memory `accessToken` via `setAccessToken`/`getAccessToken`; `credentials:'include'`;
  204 handling; `Paginated<T>` with `meta` (total/page/limit).
- **Routing** (`src/app/router.ts`): `createWebHistory`; three sections, each under
  its own layout — public (`PublicLayout` → `/`), auth (`AuthLayout` → `/sign-in`,
  `/sign-up`, `/forgot-password`, `/reset-password`, `/two-factor`, `/verify-email`),
  dashboard (`DashboardLayout`, `meta.requiresAuth`). A `beforeEach` guard redirects
  unauthenticated users (no stored access token) to `sign-in`.
- **Layout shells** (`src/layout/`): `public-layout.vue` (+ `public-header`,
  `public-footer`), `auth-layout.vue`, `dashboard-layout.vue` (+ `app-shell`,
  `app-header`/`dashboard-header`, `app-sidebar`, `user-nav`), `coming-soon.vue`.
- **i18n** (`src/shared/i18n/`): `createI18n` Composition mode (`legacy:false`),
  locales `en`/`ru`/`uz` from `messages/*.json`, `routing.ts` default locale;
  `locale-switch.vue` widget.
- **Theme & tokens** (`src/styles/globals.css`): OKLCH CSS variables in `:root`/`.dark`
  mapped by Tailwind v4 `@theme inline` to utilities (`bg-background`,
  `text-foreground`, …); `--radius`, sidebar + chart tokens, font tokens.
- **Shared kit**: `shared/components/ui/*` (shadcn-vue / reka-ui primitives),
  `shared/components/widgets/*` (icons, pager, theme-toggle, sticky-header),
  `shared/hooks/*` (use-toast, use-pagination, use-permission, use-cookie,
  use-click-outside, use-mounted), `shared/libs/{store,utils,validators}.ts`.
- **Env**: `.env.example` documents `VITE_API_URL`; real values in gitignored
  `.env.local`; the `/api/v1` prefix is added by the client, not the env.
- **Build / Docker**: `pnpm build` (`vue-tsc -b && vite build` → `dist/`) is the
  typecheck+build gate; root `Dockerfile` builds static assets served by
  `nginx:1.27-alpine` with SPA fallback (`nginx.conf`); compose files under
  `docker/compose/`.
- Out of scope: any auth/domain logic, real endpoints wiring, tests (FEAT-08).

## Consumed API

- None directly. Establishes the typed `api` client and `VITE_API_URL` base that
  all later features call (`/api/v1/...`).

## UI / components used

- `App.vue`, `HomeView.vue`, `DashboardView.vue`; layout shells; `theme-toggle`,
  `locale-switch`; shadcn-vue primitives under `shared/components/ui/`.

## Acceptance criteria

- [ ] `src/main.ts` boots `createApp` with `providers` (Pinia + i18n + Vue Query) and `router`, mounting `#app`; `globals.css` imported.
- [ ] Typed `api` client exposes envelope unwrap, `ApiError`, in-memory bearer, `getPaginated` with `meta`, 204 handling; base URL read from `VITE_API_URL`.
- [ ] Router defines public/auth/dashboard sections under their layouts; `beforeEach` redirects unauthenticated users from `requiresAuth` routes to `sign-in`.
- [ ] Theme provider flips `.light`/`.dark` on `<html>`, persists to `ui-theme`, defaults to `system`; all visuals use tokens (no inline hex).
- [ ] vue-i18n configured (Composition mode, en/ru/uz); `locale-switch` toggles instantly.
- [ ] `.env.example` documents `VITE_API_URL` only; `.env.local` gitignored; no secrets committed.
- [ ] `Dockerfile` builds static assets, served by `nginx:alpine` with SPA fallback; `pnpm build` produces `dist/`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
