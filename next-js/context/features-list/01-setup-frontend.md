# Feature 01 — Frontend Setup & Scaffold

- Feature ID: FEAT-01
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: none

> Source of truth for the **frontend foundation** — the FEAT-01 of this Next.js web
> client. It stands up the runnable App-Router shell, the typed envelope API client,
> i18n, theming, providers, route groups, and the auth guard. It writes **no auth /
> business logic** — the identity feature + auth pages land in FEAT-02; the domain
> slices (analytic/audit/payment/notification/message) follow. `## Tasks` below is
> left for `/parse-features`.

## Goal

A production-grade Next.js 16 (App Router) + React 19 + TypeScript foundation that
boots clean, talks to the backend monolith through one typed envelope client, is
localized (`en`/`ru`/`uz`), themed via Tailwind v4 tokens with light/dark, and
guards its `(protected)` routes — so feature slices can be built on a hardened base.

## Scope

- **Stack (locked):** Next.js 16 App Router + React 19, TypeScript 5 strict
  (`moduleResolution: bundler`, `@/*` → `src/*`), Tailwind CSS v4 (`@theme inline`
  in `src/styles/globals.css`), `@base-ui/react` shadcn-style primitives, TanStack
  Query 5, Zustand 5, Zod 4, `next-intl`, `next-themes`, `lucide-react`, `recharts`,
  `sonner`, `clsx`+`tailwind-merge`, `date-fns`, `class-variance-authority`. Package
  manager **pnpm** (single `pnpm-lock.yaml`; CI uses pnpm). Build target: standalone Node server
  (`output: 'standalone'`).
- **Env:** `NEXT_PUBLIC_API_URL` (backend base, default `http://localhost:3000`,
  trailing slash stripped) consumed by `shared/libs/api.ts`. Secrets live only in
  gitignored `.env.local`; `.env.example` is the sanitized reference.
- **Typed API client** (`src/shared/libs/api.ts`): one `fetch` wrapper over the
  response envelope — unwraps `data`, returns `{ data, meta }` for paginated lists
  via `getPaginated`, throws a typed `ApiError(status, code, message, fields)` on
  failure, `204` → `data: undefined`. Attaches `Authorization: Bearer <token>` from
  an in-memory token (`setAccessToken`/`getAccessToken`) and sends
  `credentials: 'include'`. Token refresh/persistence wiring is consumed by the
  identity service in FEAT-02; this feature ships the bearer + envelope primitives.
- **i18n:** `next-intl` routing (`src/shared/i18n/routing.ts`) — locales
  `['en','ru','uz']`, `defaultLocale: 'en'`, `localePrefix: 'always'`; `navigation.ts`
  (locale-aware `Link`/`redirect`), `request.ts`, message catalogs
  `messages/{en,ru,uz}.json`; middleware `src/proxy.ts` for locale routing.
- **Theme/tokens:** Tailwind v4 `@theme inline` token set (oklch CSS vars, light
  `:root` + `.dark` overrides) in `src/styles/globals.css`; `next-themes`
  `ThemeProvider` (`attribute="class"`, `defaultTheme="system"`); `theme-toggle`
  + `locale-switch` widgets.
- **App Router shell + route groups:** `src/app/layout.tsx` (root html),
  `src/app/[locale]/layout.tsx` (locale provider wrapper), route groups
  `(auth)`, `(protected)`, `(public)` each with their own `layout.tsx`; the
  `(protected)` layout renders the dashboard shell (`src/layout/dashboard-layout.tsx`).
- **Auth guard:** the `(protected)` layout redirects unauthenticated users to the
  `(auth)` sign-in route (client-side guard reading the persisted access token).
- **Providers:** `src/shared/components/provider/providers.tsx` composing
  `I18nProvider` → `ThemeProvider` → `QueryProvider`.
- **Out of scope:** auth/identity logic & pages (FEAT-02), any domain feature
  (FEAT-03…07), the marketing landing content (FEAT-09), tests (FEAT-08).

## Consumed API

- Envelope only: success `{ success, data, meta? }`, error
  `{ success, error: { code, message, fields? } }`; `meta: { total, page, limit }`
  on lists. No specific endpoints are called by this feature — it ships the client
  that every feature's `api/` folder uses (base `${NEXT_PUBLIC_API_URL}/api/v1`).

## UI / components used

- `src/styles/globals.css` (token source); `shared/components/provider/*`
  (query/theme/i18n providers); `shared/components/widgets/{locale-switch,theme-toggle,icons,sticky-header,pager}`;
  `shared/components/ui/*` (shadcn-on-`@base-ui/react` primitives — button, card,
  input, etc.); `src/layout/dashboard-layout.tsx` + chrome (`app-sidebar`,
  `dashboard-header`, `user-nav`); `shared/libs/{api,store,validators,utils}.ts`;
  `shared/hooks/*` (`use-permission`, `use-pagination`, `use-mobile`, `use-toast`, …).

## Acceptance criteria

- [ ] App boots: `/` redirects to a locale-prefixed route (`/en`), which returns 200.
- [ ] Typed client unwraps the envelope, returns `{ data, meta }` for paginated
      lists, throws `ApiError` (with `code`/`fields`) on failure, and `204` → `undefined`.
- [ ] Bearer header attached from in-memory token via `setAccessToken`; requests send
      `credentials: 'include'`; base URL read from `NEXT_PUBLIC_API_URL` (slash stripped).
- [ ] All three locales (`en`/`ru`/`uz`) resolve with `localePrefix: 'always'`; locale
      switch + message catalogs work; user-facing strings go through `next-intl`.
- [ ] Light/dark theme toggles via `next-themes`; all visual values resolve to tokens
      from `globals.css` (no hardcoded hex/px).
- [ ] Route groups `(auth)`/`(protected)`/`(public)` render with their layouts;
      `(protected)` redirects unauthenticated users to `(auth)` sign-in.
- [ ] Providers compose i18n → theme → query around the app.
- [ ] `.env.example` holds only the API URL; `.env.local` gitignored; no secrets committed.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
