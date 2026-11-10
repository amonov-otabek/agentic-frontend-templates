# Feature 02 — Identity & Auth (Frontend)

- Feature ID: FEAT-02
- Status: draft
- Depends on: FEAT-01 (scaffold: API client, providers, router, layouts, theme, i18n)

> Source of truth for the **frontend identity layer**: the full auth surface, session +
> token handling, profile, and the RBAC admin user-management surface. It wires the
> `features/identity/*` modules to the backend identity API; it builds no backend.

## Goal

Make sign-in / sign-up and the full identity surface work end-to-end against the backend
`/api/v1` identity API: token persistence with refresh rotation, guarded dashboard
routes, email verification, password reset, 2FA, profile/settings, and an admin
user-management surface gated by role.

## Scope

- **Auth pages** (`AuthLayout` group, `features/identity/pages/*`):
  - `sign-in-page` — identifier (email/phone) + password, `remember_me`, `trust_device`;
    on `requires_two_fa` stash the pending token and route to `/two-factor`.
  - `sign-up-page` — email/phone + password + names, terms/privacy acceptance.
  - `verify-email-page` — consume an email-link token (`/auth/verify-email-link`).
  - `forgot-password-page` — request a reset email.
  - `reset-password-page` — submit token + new password.
  - `two-factor-page` — complete the 2FA challenge with the pending token.
- **Settings page** (`features/identity/pages/settings-page` under `DashboardLayout`,
  `/settings`) — profile edit, password change, 2FA setup/enable/disable, active-session
  list + revoke / revoke-all.
- **Admin users page** (`features/identity/pages/users-page`, `/users`) — paginated user
  list, detail, toggle-active, role assignment, delete; roles list. Gated by role via
  `shared/hooks/usePermission`.
- **Session / token handling** (`services/auth.service.ts`) — persist access + refresh in
  `localStorage`, hydrate via `loadSession()`, in-memory bearer; refresh-token rotation;
  forced sign-out (clear tokens + redirect `/sign-in`) on `401/403`; pending-2FA token in
  `sessionStorage`.
- **Client auth state** (`store/auth.store.ts`, Zustand) — `user`, `status`, `setUser`,
  `clear`; `useMe` hydrates the store on load; route guard reads `isAuthenticated()`.
- **Validators** (`validators/auth.validator.ts`, Zod) — sign-in/up, forgot/reset,
  2FA, profile, change-password; server field errors (`error.fields`) map onto inputs.
- **Queries** (`queries/{auth,settings,admin}.queries.ts`) — `useMe`, `useSignIn`,
  `useSignUp`, `useSignOut`, `useVerifyTwoFactor`, `useForgotPassword`, `useResetPassword`,
  `useVerifyEmail`, profile/password/2FA/session hooks, admin user hooks; mutations
  invalidate the relevant query keys.
- Out of scope: OAuth providers, devices/security center, and address book (add as token
  follow-ups only if the backend exposes them — not in the base template).

## Consumed API

Base `/api/v1`, envelope `{ success, data, meta? }`. Branch on `error.code` / `error.fields`.

- **Auth:** `POST /auth/sign-in` (2FA → `requires_two_fa` + `pending_token`, else `tokens`)
  · `POST /auth/sign-up` · `POST /auth/refresh` (rotate pair) · `POST /auth/logout`.
- **Verify / password:** `POST /auth/verify-email-link` · `POST /auth/forgot-password`
  · `POST /auth/reset-password` · `PATCH /auth/password`.
- **2FA:** `GET /2fa/status` · `POST /2fa/setup` (secret + QR) · `POST /2fa/enable`
  (backup codes) · `POST /2fa/verify` (challenge → tokens) · `DELETE /2fa/disable`.
- **Profile:** `GET /profile/me` · `PATCH /profile/me`.
- **Sessions:** `GET /sessions` · `DELETE /sessions/:id` · `DELETE /sessions`.
- **Admin (role-gated):** `GET /admin/users` (paginated) · `GET /admin/users/:id`
  · `PATCH /admin/users/:id/toggle-active` · `PATCH /admin/users/:id/role` ·
  `DELETE /admin/users/:id` · `GET /admin/roles`.

## UI / components used

- `features/identity/components/*`: `sign-in-form`, `sign-up-form`, `two-factor-form`,
  `forgot-password-form`, `reset-password-form`, `verify-email-view`, `auth-banner`,
  `profile-form`, `password-form`, `two-factor-card`, `sessions-table`, `users-table`,
  `roles-table`.
- shadcn primitives (`button`, `card`, `input`, `field`, `label`, `tabs`, `table`,
  `dialog`, `alert-dialog`, `badge`, `input-otp`, `pagination`, `sonner`), `shared/hooks`
  (`usePermission`, `usePagination`, `use-toast`), `AuthLayout` / `DashboardLayout`.

## Acceptance criteria

- [ ] Sign-up and sign-in work against the live API; access token in memory, refresh
      persisted in `localStorage` and rehydrated on startup.
- [ ] Refresh rotation works: expired access → silent refresh; revoked/invalid refresh →
      forced sign-out and redirect to `/sign-in`.
- [ ] On `requires_two_fa`, the pending token is stashed and `/two-factor` completes auth.
- [ ] Protected dashboard routes redirect unauthenticated users to `/sign-in`;
      authenticated users skip the auth group.
- [ ] Email-link verification, forgot → reset, and change-password flows work.
- [ ] 2FA full cycle: status → setup (QR) → enable (backup codes) → sign-in challenge
      verify → disable.
- [ ] Settings page edits profile, lists + revokes sessions (single + all), and manages 2FA.
- [ ] Admin users page lists (paginated), shows detail, toggles active, assigns a role,
      and deletes — gated by `usePermission`; non-admins cannot reach it.
- [ ] Server field errors (`error.fields`) map onto the right form inputs; assertions
      branch on `error.code`, never message text.
- [ ] All copy + validation errors localized (en/ru/uz); auth state in the Zustand store;
      mutations invalidate query keys.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
