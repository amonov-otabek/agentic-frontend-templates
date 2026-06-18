# Feature 02 — Identity & Auth (Frontend)

- Feature ID: FEAT-02
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01 (frontend foundation: api.service, interceptors, guards, theme, i18n)

> Source of truth for the **frontend identity layer**. This project is **empty
> (no code exists yet)** — everything below is **TO BUILD**. `/parse-features 02`
> turns the `## Tasks` section into the per-feature task file. This is the **largest**
> spec: it builds the full auth surface, session/token handling, profile, and
> RBAC route guards against the backend `identity` domain.

## Goal

Make sign-up / sign-in and the full identity surface **work end-to-end** in the
Angular SPA against the backend `/api/v1` identity endpoints: account creation,
verification, password recovery, two-factor, session/device management, profile,
role-switching, and an admin user-management area — all guarded, typed, themed, and
localized.

## Scope

### Routes / pages (`features/identity/`)

A lazy `identity.routes.ts` mounting eight pages, split across a public auth group
and a protected/admin group:

| Page | Route | Group | Purpose |
| --- | --- | --- | --- |
| Sign-up | `/auth/sign-up` | public | email/phone + password registration |
| Sign-in | `/auth/sign-in` | public | login (2FA-enabled accounts get a challenge, not tokens) |
| Verify-email | `/auth/verify-email` | public | code + magic-link verification, resend (rate-limited) |
| Forgot-password | `/auth/forgot-password` | public | request reset link/code |
| Reset-password | `/auth/reset-password` | public | set new password via token |
| Two-factor | `/auth/two-factor` | public/protected | TOTP challenge on sign-in + setup/enable/disable |
| Settings | `/account/settings` | protected | profile + security center (password, 2FA, sessions, devices, linked accounts) |
| Admin users | `/admin/users` | admin | user list/search, detail, toggle-active, role assign, delete |

### Components (`features/identity/components/`)

`sign-in-form` · `sign-up-form` · `verify-email-view` · `forgot-password-form` ·
`reset-password-form` · `two-factor-form` (challenge + setup QR/verify) ·
`profile-form` · `password-form` · `sessions-list` · `devices-list` ·
`linked-accounts` · `auth-banner`/auth shell · admin
`users-table` + `user-detail`. Presentational only — they take inputs / emit outputs;
pages own state and call services.

### Services / state (`features/identity/services/` + `core/auth/`)

- `auth.service` (core) — token store via signals (`accessToken` in memory, refresh
  per strategy), `currentUser` signal, `status` signal (`authenticated`/`guest`/`loading`),
  `setUser`/`clear`, session hydration on app launch.
- Feature services wrapping the core `api.service`: `auth.api` (sign-up/in/refresh/logout),
  `verify.api`, `password.api`, `twofa.api`, `session.api`, `device.api`, `oauth.api`,
  `profile.api`, `admin.api`. Components never call `HttpClient`.
- Reactive Forms + typed validators (`validators/auth.validators.ts`) for every form,
  mirroring backend validation and surfacing `error.fields` on submit failure.

### Guards / interceptors

- `auth.guard` — protected routes require an authenticated session; else redirect to
  `/auth/sign-in`; authenticated users are bounced out of the public auth group.
- `role.guard` — `/admin/**` requires the `admin` role claim; `403` otherwise.
- Relies on FEAT-01's `auth.interceptor` for bearer attach + `401` refresh-rotation
  and `error.interceptor` for typed errors.

## Consumed API

All under base `/api/v1`, response envelope `{ success, data, error, meta }`.

- **Auth:** `POST /auth/sign-up` · `POST /auth/sign-in` (2FA-enabled → challenge) ·
  `POST /auth/refresh` (rotate pair) · `POST /auth/logout`.
- **Verify:** `POST /auth/verify-email` (code) · `POST /auth/verify-email-link` (magic
  link) · `POST /auth/verify-phone` · `POST /auth/resend-verification`.
- **Password:** `POST /auth/forgot-password` · `POST /auth/reset-password` ·
  `PATCH /auth/password`.
- **2FA (TOTP):** `GET /2fa/status` · `POST /2fa/setup` (secret + QR) ·
  `POST /2fa/enable` · `POST /2fa/verify` (challenge → tokens) · `DELETE /2fa/disable`.
- **OAuth:** `GET /oauth/providers` · `POST /oauth/callback` (guarded) ·
  `DELETE /oauth/:provider`.
- **Sessions / devices / security:** `GET|DELETE /sessions` · `DELETE /sessions/:id` ·
  `GET /devices` · `PATCH /devices/:id/trust` · `DELETE /devices/:id` · `GET /security`.
- **Profile:** `GET|PATCH /profile/me` · `POST /profile/avatar` (avatar upload) ·
  `GET /profile/:id` (public profile).
- **Admin (`admin` role):** `GET /admin/users` (search) · `GET /admin/users/:id` ·
  `PATCH /admin/users/:id/toggle-active` · `PATCH /admin/users/:id/role` ·
  `DELETE /admin/users/:id` · `GET /admin/roles`.

Every consumed endpoint is registered in `context/api-context.md`'s endpoint registry.

## UI / components used

- Shared primitives from FEAT-01 (button, card, input, badge/chip) plus an auth shell
  layout; forms use Reactive Forms with token-only styling and full state coverage
  (default/hover/focus-visible/disabled/loading; loading/empty/error/success on async).
- All copy + validation errors localized via the i18n service; theme tokens only.

## Acceptance criteria

- [ ] `features/identity/api/` matches the canonical `/api/v1` identity contract above
      (paths + envelope), registered in `context/api-context.md`.
- [ ] Sign-up (email/phone + password) and sign-in work against the backend; access
      token in memory, refresh persisted per strategy; tokens never logged.
- [ ] Refresh rotation works: expired access → silent refresh-and-retry; revoked
      refresh → forced sign-out + redirect to `/auth/sign-in`.
- [ ] Protected routes guarded by `auth.guard`; admin routes guarded by `role.guard`;
      authenticated users skip the public auth group.
- [ ] Email verification (code + magic link) and phone verification work; resend
      respects rate limit and surfaces `429` cleanly.
- [ ] forgot → reset → change-password flows work.
- [ ] 2FA full cycle (setup → enable → sign-in challenge verify → disable); status
      reflected in the security center.
- [ ] OAuth sign-in lists providers and supports link/unlink.
- [ ] Profile edit (name, avatar, language) via `GET|PATCH /profile/me` +
      `POST /profile/avatar`; public profile read via `GET /profile/:id`.
- [ ] Admin users area: list/search, detail, toggle-active, role assign
      (`PATCH /admin/users/:id/role`), delete — `admin` only, guarded.
- [ ] All forms use Reactive Forms + typed validators; server `error.fields` surfaced
      on submit failure; UI branches on `error.code`, never message text.
- [ ] All auth copy + errors localized; theme tokens applied; OnPush everywhere.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
