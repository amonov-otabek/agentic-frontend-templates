# Project Overview

A concise, high-level summary of this project. Every agent reads this before any
feature work. This is the **React (Vite) SPA web client** for the
`express-monolith-api` backend.

## What we are building

A production-ready **Vite + React 19 + TypeScript single-page application**. A static
bundle (built by Vite, served by `nginx:alpine` with SPA fallback) that renders the
web surface for the backend monolith: a public marketing area, an auth flow, and a
guarded dashboard with one feature section per backend domain module (identity,
message, notification, payment, audit, analytic).

It is a pure client — no server runtime of its own. All data and identity come from
the backend HTTP API (`/api/v1/*`); the SPA holds an in-memory bearer token and talks
to the API through a single typed `fetch` client.

## Goals

1. Ship a hardened frontend foundation: routing, auth + route guards, server-state
   data fetching, a token-driven design system, theming, i18n, and accessible
   components — production-grade from the first commit.
2. Mirror the backend's domain modules as isolated feature folders so each surface can
   be built, tested, and reasoned about independently while shipping as one bundle.

## Surfaces

| Surface | Layout | Who uses it |
| --- | --- | --- |
| Public site | `PublicLayout` (`/`) | anonymous visitors |
| Auth flow | `AuthLayout` (`/sign-in`, `/sign-up`, `/forgot-password`, `/reset-password`, `/two-factor`, `/verify-email`) | signing-in users |
| Dashboard | `DashboardLayout` (`/dashboard`, `/messages`, `/users`, `/notifications`, `/audit`, `/analytics`, `/payments`, `/settings`) | authenticated users |

## Core flows

1. A visitor signs in (email/phone + password, optional 2FA challenge); the auth
   service persists the access + refresh token pair and sets the in-memory bearer
   token used by the API client.
2. Guarded dashboard routes check for a stored session; unauthenticated users are sent
   to `/sign-in`. Authenticated requests carry `Authorization: Bearer <token>`.
3. Pages read server state via TanStack Query hooks (`useQuery` / `useMutation`) that
   call feature API modules; mutations invalidate the relevant query keys to refetch.
4. Responses arrive in the backend envelope (`{ success, data, meta? }`); the API
   client unwraps `data` (and `meta` for paginated lists) or throws a typed `ApiError`.

## Roles

| Role | Description |
| --- | --- |
| user | standard authenticated account; sees the dashboard sections |
| admin | elevated access (e.g. the `/users` admin surface); gated by `use-permission` |
| _(RBAC)_ | roles/permissions come from the backend; the client reflects them, never grants them |

## Key constraints

- The client never trusts itself for authorization — it reflects backend role checks;
  guards are UX, not security.
- Every visual value comes from a design token (`context/ui-context.md`); zero hardcoded
  colors, sizes, or spacing.
- All user-facing strings go through i18n (`react-i18next`, `en` / `ru` / `uz`).
- Built as a static bundle; no secrets in the client — only `VITE_API_URL` (public).

## In scope (this milestone)

- Feature areas become `context/features-list/` specs and build per backend module.
  Identity, message, audit are the reference-complete surfaces; notification,
  analytic, payment have scaffolds to fill in.

## Out of scope

- Backend/API implementation (owned by `express-monolith-api`; this client only
  consumes it — see `context/api-context.md`).
- Server-side rendering and any server runtime (this is a static Vite SPA; the
  `next-js` template covers SSR).
