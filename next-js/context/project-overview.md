# Project Overview

A concise, high-level summary of this project. Every agent reads this before any
feature work. This is the **Next.js web client** for the backend monolith API.

## What we are building

A production-ready **Next.js 16 + React 19 + TypeScript web client** that consumes the
backend monolith API (`/api/v1`). A single App-Router application with a localized,
feature-based structure (identity, message, notification, payment, audit, analytic),
served as a standalone Node server. It mirrors the backend's domain modules surface-for-surface.

## Goals

1. Ship a hardened frontend foundation: auth flows, route guards, data fetching with
   cache + invalidation, forms with shared validation, theming, and i18n — production-grade
   from the first commit.
2. Keep feature slices isolated (`features/<name>/{api,services,queries,store,components,pages,…}`)
   so they can be developed, tested, and reasoned about independently while shipping as one app.

## Platforms

| Surface | Tech | Who uses it |
| --- | --- | --- |
| Web app | Next.js 16 App Router (`/[locale]/…`) | end users, admins |
| Auth area | `(auth)` route group | unauthenticated visitors |
| Dashboard | `(protected)` route group | authenticated users / admins |
| Marketing | `(public)` route group | anonymous visitors |
| Backend it consumes | REST API at `NEXT_PUBLIC_API_URL` (`/api/v1`) | this client |

## Core flows

1. A visitor signs in (email + password, with optional 2FA) via the `(auth)` group;
   the auth service stores the JWT access + refresh pair and primes the in-memory bearer token.
2. Authenticated navigation enters the `(protected)` group; the layout guard checks the
   session and redirects unauthenticated users to sign-in.
3. Pages render feature `pages/` which compose feature `components/`; data is fetched
   through TanStack Query hooks (`queries/`) that call the typed API client (`api/`).
4. Mutations (compose message, create payment, update profile) call the API client,
   then invalidate the relevant query keys so the UI re-fetches fresh state.

## Roles

| Role | Description |
| --- | --- |
| user | standard authenticated account |
| admin | elevated access to admin surfaces (users, audit) |
| _(RBAC)_ | role/permission gating mirrors the backend; `use-permission` hook drives conditional UI |

## Key constraints

- The backend response envelope is the contract: `{ success, data, meta? }` /
  `{ success, error: { code, message, fields? } }`. Assert on `error.code`, not message text.
- All visual values come from design tokens in `context/ui-context.md` — no hardcoded colors/sizes.
- All user-facing strings go through `next-intl` (locales: `en`, `ru`, `uz`; prefix always).
- Image strategy: Next standalone Node server (`output: 'standalone'`).

## In scope (this milestone)

- Feature areas become `context/features-list/` specs and `context/tasks-list/` task files.
  Identity is the reference slice; the remaining domain slices follow its shape.

## Out of scope

- The backend API itself (built and owned separately — see `context/api-context.md` for
  the contract this client depends on).
- Infrastructure provisioning beyond the provided Docker build.
