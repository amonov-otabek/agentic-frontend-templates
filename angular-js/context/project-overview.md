# Project Overview

A concise, high-level summary of this project. Every agent reads this before any
feature work. This is the **planned foundation for an empty project** — no code
exists yet; the stack and flows below are the proposed, locked starting point.

## What we are building

A production-grade **Angular single-page web client** (latest Angular, standalone
components, no NgModules). It is a sibling to the `next-js`, `react-js`, and `vue-js`
SPA templates and consumes the **same backend API** (the `express-monolith-api`
modular monolith). The app is built static via the Angular CLI and served by
`nginx:alpine` with SPA fallback, exactly like the Vite-based SPA siblings.

## Goals

1. Ship a hardened frontend foundation: routing, auth/JWT against the backend,
   typed API access, a token-driven design system, and accessibility baked in —
   production-grade from the first commit.
2. Keep features isolated in feature folders so they can be developed, tested, and
   reasoned about independently while shipping as one SPA bundle.

## Platforms / surfaces

| Surface | Tech | Who uses it |
| --- | --- | --- |
| Web SPA | Angular (standalone) + Angular Router | end users in a browser |
| Static host | `nginx:alpine` (SPA fallback) | production delivery |
| Backend API | `express-monolith-api` (`/api/v1`) | consumed over HTTPS |
| Dev server | `ng serve` (HMR) | developers |

## Core flows

1. A visitor lands on a public route (login, register, marketing). Unauthenticated
   access to a protected route is redirected to login by a route guard.
2. A user authenticates (email/phone + password, OAuth, or 2FA) via the backend and
   receives a JWT access + refresh token pair, stored and managed by an auth service.
3. Authenticated requests go through a typed API service; an HTTP interceptor attaches
   the bearer token and, on `401`, attempts a refresh-and-retry before redirecting.
4. Feature pages (smart components) call services, render state via signals, and
   compose presentational components from the design system.

## Roles

| Role | Description |
| --- | --- |
| guest | unauthenticated visitor — public routes only |
| user | standard authenticated account |
| admin | elevated access; admin-only routes gated by a role guard |
| _(RBAC)_ | role/permission claims come from the backend; guards extend per project |

## Key constraints

- Consumes the backend response envelope verbatim — assert on `error.code`, never
  message text (the backend localizes messages via i18n).
- Tokens never logged; access token kept in memory where possible, refresh handled
  by the interceptor; never embed secrets in the bundle.
- TypeScript strict; no visual value outside the design tokens (`context/ui-context.md`).
- WCAG-minded: keyboard reachable, visible focus, contrast ≥ 4.5:1.

## In scope (this milestone)

- Feature areas become `context/features-list/` specs. The first is **01-setup-frontend**
  (Angular CLI app + routing + HttpClient + interceptors + design tokens + Docker/nginx
  build). Auth and domain features follow.

## Out of scope

- The backend itself (a separate project — see `context/api-context.md` for the
  contract this client consumes).
- Native/mobile shells and SSR/Angular Universal (this is a static-built SPA).
