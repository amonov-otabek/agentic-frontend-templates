# Project Overview

A concise, high-level summary of this project. Every agent reads this before any
feature work. Derived from the `vue-js` reference template.

## What we are building

A production-ready **Vite + Vue 3 + TypeScript single-page web client**. A static SPA
that surfaces the backend's domain features (identity, messaging, notification,
payment, audit, analytics) to end users — built once with Vite and served as static
assets by `nginx:alpine` with a client-side routing fallback. It talks to the
`express-monolith-api` over HTTP using a typed fetch client and the shared response
envelope.

## Goals

1. Ship a hardened frontend foundation: auth flows (sign-in/up, 2FA, password reset,
   email verify), guarded dashboard, server-state caching, i18n, theming, and a
   complete shadcn-vue component library — production-grade from the first commit.
2. Keep feature folders isolated so each domain (identity, message, audit, …) can be
   developed, tested, and reasoned about independently while shipping as one bundle.

## Surfaces

| Surface | Tech | Who uses it |
| --- | --- | --- |
| Public site | Vue Router `PublicLayout` (`/`) | anonymous visitors |
| Auth pages | `AuthLayout` (`/sign-in`, `/sign-up`, `/forgot-password`, `/two-factor`, …) | signing-in users |
| Dashboard | `DashboardLayout` (`/dashboard`, `/messages`, `/users`, …) — `requiresAuth` | authenticated users |
| Backend API | `express-monolith-api` (`/api/v1`) via typed `api` client | consumed, not hosted |

## Core flows

1. A visitor signs in (email/phone + password, or 2FA challenge); the auth service
   persists the JWT access + refresh token pair (localStorage) and seeds the in-memory
   bearer token used by the API client.
2. A `vue-router` `beforeEach` guard gates `requiresAuth` routes — no stored session
   redirects to `/sign-in`.
3. Pages mount feature components; components call TanStack Query hooks
   (`queries/*.queries.ts`) which call the feature API layer (`api/*.api.ts`) over the
   shared envelope. Server state lives in the query cache; cross-cutting client state
   (e.g. current user) lives in a Pinia store.
4. The API client attaches the bearer token, unwraps the envelope, and throws a typed
   `ApiError` on failure; UI surfaces field errors and toasts.

## Roles

| Role | Description |
| --- | --- |
| visitor | anonymous, public pages only |
| user | standard authenticated account; dashboard access |
| admin | elevated access (users/roles admin pages) |

The backend's `roles`/`permissions` drive access; the client mirrors them via
`use-permission` and route guards. Treat the backend as the authority.

## Key constraints

- All visual values come from design tokens (`src/styles/globals.css`) — never inline.
- All server state goes through TanStack Query; no ad-hoc `fetch` in components.
- All user-facing strings resolve through `vue-i18n` (en / ru / uz).
- The build output is static — there is **no** server runtime (no SSR, no middleware).

## In scope (this milestone)

- Feature areas become `context/features-list/` specs and are built per-feature via
  `/parse-features` → `/start-build`. Identity is the reference; message/audit follow.

## Out of scope

- The backend itself (consumed only — see `context/api-context.md`).
- Server-side rendering / Node server (this SPA is static-served by nginx).
- Infrastructure provisioning beyond the provided Dockerfile + compose.
