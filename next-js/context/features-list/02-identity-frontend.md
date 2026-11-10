# Feature 02 — Identity & Auth (Frontend)

- Feature ID: FEAT-02
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01

> Source of truth for the **frontend identity layer** — the reference feature slice.
> It wires the full auth surface end-to-end against the backend identity domain on
> `${NEXT_PUBLIC_API_URL}/api/v1`: sign-up, sign-in (with optional 2FA), email
> verification, forgot/reset password, two-factor, account settings (profile,
> password, sessions, 2FA), and the RBAC-gated admin users area. `## Tasks` below is
> left for `/parse-features`. This is the largest slice (~33 files under
> `src/features/identity/`).

## Goal

Make sign-in / sign-up and the full identity surface work end-to-end: token
persistence + in-memory bearer, refresh re-auth, 2FA challenge hop, guarded
routes, profile & security settings, and an admin-only user-management area —
all localized and tokens-themed.

## Scope

- **Auth pages (8) under `src/app/[locale]/(auth)/` + `(protected)`:**
  `sign-up`, `sign-in`, `verify-email`, `forgot-password`, `reset-password`,
  `two-factor` (`(auth)` group), plus `settings` and admin `users` (`(protected)`
  group). Route files are thin and render `features/identity/pages/*`.
- **Session / token handling** (`services/auth.service.ts`): persist access +
  refresh to `localStorage` (`auth.access_token` / `auth.refresh_token`), prime the
  in-memory bearer via `setAccessToken`, hydrate on startup (`loadSession`), clear on
  sign-out (revokes refresh via `/auth/logout`). The 2FA challenge `pending_token` is
  stashed in `sessionStorage` across the sign-in → two-factor hop and exchanged at
  `/2fa/verify`.
- **State:** Zustand `store/auth.store.ts` (`user`, `setUser`, `clear`) with the
  auto-selector helper; query keys `authKeys.me`.
- **Queries** (`queries/{auth,settings,admin}.queries.ts`): `useMe`, `useSignIn`,
  `useSignUp`, `useSignOut`, `useVerifyTwoFactor`, `useForgotPassword`,
  `useResetPassword`, `useVerifyEmail`; settings (profile/password/sessions/2FA);
  admin (users list/detail/toggle/role/delete/roles).
- **Validators:** `validators/auth.validator.ts` (Zod: sign-in/up, forgot/reset,
  2FA, profile, password) — field errors map onto form fields via the envelope's
  `error.fields`.
- **Profile / settings hub** (`pages/SettingsPage.tsx`): edit profile
  (`profile-form`), change password (`password-form`), 2FA setup/enable/disable
  (`two-factor-card`), active sessions list + revoke (`sessions-table`).
- **RBAC-gated admin** (`pages/UsersPage.tsx`): user list/search, detail,
  toggle-active, role assign, delete, roles list — gated by the `usePermission`
  hook (admin role) and the `(protected)` guard.
- **Out of scope:** OAuth provider linking and address book (not in the template
  surface); bespoke settings redesigns.

## Consumed API

Base `${NEXT_PUBLIC_API_URL}/api/v1`, envelope `{ success, data, error, meta }`.

- **Auth:** `POST /auth/sign-up` · `POST /auth/sign-in` (2FA-enabled → `requires_two_fa` + `pending_token`, not tokens) · `POST /auth/refresh` (new pair) · `POST /auth/logout`
- **Verify / password:** `POST /auth/verify-email-link` · `POST /auth/forgot-password` · `POST /auth/reset-password`
- **2FA:** `GET /2fa/status` · `POST /2fa/setup` (secret + QR) · `POST /2fa/enable` (→ backup codes) · `POST /2fa/verify` (challenge → token pair) · `DELETE /2fa/disable`
- **Profile / sessions:** `GET /profile/me` · `GET /sessions` · `DELETE /sessions/:id` · `DELETE /sessions`
- **Admin (RBAC):** `GET /admin/users` · `GET /admin/users/:id` · `PATCH /admin/users/:id/toggle-active` · `PATCH /admin/users/:id/role` · `DELETE /admin/users/:id` · `GET /admin/roles`
- Types mirror the backend: `PublicUser`, `AuthResponse`, `TokenPair`, `SignInDto`,
  `SignUpDto`, `Session`, `TwoFaStatus`, `TwoFaSetup`, `AdminUser`, `Role`.

## UI / components used

- `features/identity/components/`: `sign-in-form`, `sign-up-form`,
  `two-factor-form`, `forgot-password-form`, `reset-password-form`,
  `verify-email-view`, `auth-banner`, `profile-form`, `password-form`,
  `two-factor-card`, `sessions-table`, `users-table`, `roles-table`.
- `(auth)/layout.tsx` auth shell; `(protected)/layout.tsx` dashboard guard shell.
- Shared UI primitives (`button`, `card`, `input`, `field`, `label`, `input-otp`,
  `table`, `dialog`, `badge`, `tabs`, `sonner`); `usePermission`, `use-toast`,
  `usePagination` hooks; `Link`/`redirect` from `shared/i18n/navigation`.

## Acceptance criteria

- [ ] Sign-up and sign-in (identifier + password) work against the live API; access
      token in memory + persisted, refresh persisted.
- [ ] 2FA: `requires_two_fa` stashes `pending_token` in `sessionStorage`; the
      two-factor page exchanges it at `/2fa/verify` for the token pair.
- [ ] Refresh re-auth on reload via `loadSession`; sign-out revokes the refresh token
      and clears storage + query cache.
- [ ] `(protected)` routes redirect unauthenticated users to sign-in; authenticated
      users reach the dashboard.
- [ ] Email verification (link), forgot → reset password flows work; resend respects
      backend rate limits (surfaced via `error.code`).
- [ ] Settings: profile edit persists; password change works; 2FA setup → enable
      (backup codes shown) → disable cycle; sessions list + revoke / revoke-all.
- [ ] Admin users area gated by `usePermission` (admin) + the route guard: list,
      detail, toggle-active, role assign, delete, roles list.
- [ ] Zod field errors map onto form fields; assertions branch on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
