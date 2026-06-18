# Feature 02 — Identity & Auth (Frontend)

- Feature ID: FEAT-02
- Status: draft
- Depends on: FEAT-01 (scaffold: API client, router, layouts, providers, i18n, theme)

> Source of truth for the **frontend identity layer**: the full auth surface,
> session/token handling, profile, the security center, and RBAC admin pages.
> `/parse-features 02` turns the `## Tasks` section into
> `context/tasks-list/02-identity-frontend-tasks.md`. This is the reference
> feature — message/audit/analytic/payment/notification follow its shape.

## Goal

Make the complete identity surface work end-to-end against the backend identity
domain on `VITE_API_URL`: sign-up / sign-in (with optional 2FA challenge), email
verification, forgot / reset password, the TOTP two-factor cycle, profile +
security settings (sessions/devices/2FA/password), and `ADMIN`-only user
management — all token-secured with refresh-rotation, route-guarded, themed, and
localized (en/ru/uz).

## Scope (real pages, routes, components, composables, stores)

- **Pages** (`src/features/identity/pages/`, mounted in `src/app/router.ts`):
  - `(auth)` group under `AuthLayout`: `sign-in-page.vue` (`/sign-in`),
    `sign-up-page.vue` (`/sign-up`), `forgot-password-page.vue`
    (`/forgot-password`), `reset-password-page.vue` (`/reset-password`),
    `two-factor-page.vue` (`/two-factor`), `verify-email-page.vue`
    (`/verify-email`).
  - `(dashboard)` group under `DashboardLayout` (`requiresAuth`):
    `settings-page.vue` (`/settings`), `users-page.vue` (`/users`, admin).
- **Components** (`src/features/identity/components/`): `sign-in-form.vue`,
  `sign-up-form.vue`, `forgot-password-form.vue`, `reset-password-form.vue`,
  `two-factor-form.vue`, `verify-email-view.vue`, `auth-banner.vue`,
  `profile-form.vue`, `password-form.vue`, `two-factor-card.vue`,
  `sessions-table.vue`, `users-table.vue`, `roles-table.vue`.
- **Validators** (`validators/auth.validator.ts`, Zod via `@vee-validate/zod`):
  `signInSchema` (identifier + password + remember_me/trust_device), `signUpSchema`
  (name + email + password + terms accept), `forgotPasswordSchema`,
  `resetPasswordSchema`, `twoFactorSchema` (6-digit TOTP or 8-char backup),
  `updateProfileSchema`, `changePasswordSchema`.
- **Services** (`services/auth.service.ts`): token persist/clear (access +
  refresh in localStorage, in-memory bearer via `setAccessToken`), `loadSession`
  hydrate-on-import, `getRefreshToken`, `isAuthenticated` (used by the router
  guard), pending-2FA stash in `sessionStorage`, `signIn`/`signUp`/
  `verifyTwoFactor`/`signOut` orchestration.
- **Store** (`store/auth.store.ts`, Pinia setup store): `user`, `status`
  (`idle`/`authenticated`/`unauthenticated`), `isAuthenticated`, `setUser`, `clear`.
- **Queries** (`queries/`): `auth.queries.ts` (`useMe`, `useSignIn`, `useSignUp`,
  `useSignOut`, `useVerifyTwoFactor`, `useForgotPassword`, `useResetPassword`,
  `useVerifyEmail`), `settings.queries.ts` (`useUpdateProfile`, `useChangePassword`,
  `useSessions`, `useRevokeSession`, `useRevokeAllSessions`, `useTwoFaStatus`,
  `useSetupTwoFa`, `useEnableTwoFa`, `useDisableTwoFa`), `admin.queries.ts`
  (`useAdminUsers`, `useRoles`, `useToggleUserStatus`, `useAssignRole`,
  `useDeleteUser`).
- **Types** (`types/identity.types.ts`): `PublicUser`, `TokenPair`, `AuthResponse`,
  `SignInDto`, `SignUpDto`, `UpdateProfileDto`, `ChangePasswordDto`, `Session`,
  `TwoFaStatus`, `TwoFaSetup`, `AdminUser`, `Role`.
- **RBAC**: `use-permission` + `user.role` from the store gate the admin
  `/users` route/page; backend roles/permissions are authoritative.
- Out of scope: dedicated OAuth provider screens and address book (not in the
  template API) — flag if scoped later.

## Consumed API (`/api/v1`, shared envelope)

- **Auth** (`api/auth.api.ts`): `POST /auth/sign-in`, `POST /auth/sign-up`,
  `POST /auth/refresh` (rotate pair), `POST /auth/logout`, `GET /profile/me`,
  `POST /auth/forgot-password`, `POST /auth/reset-password`,
  `POST /2fa/verify`, `POST /auth/verify-email-link`.
- **Profile** (`api/profile.api.ts`): `PATCH /profile/me`, `PATCH /auth/password`.
- **Sessions** (`api/session.api.ts`): `GET /sessions`, `DELETE /sessions/:id`,
  `DELETE /sessions`.
- **2FA** (`api/twofa.api.ts`): `GET /2fa/status`, `POST /2fa/setup`,
  `POST /2fa/enable`, `DELETE /2fa/disable`.
- **Admin** (`api/admin.api.ts`): `GET /admin/users`, `GET /admin/users/:id`,
  `PATCH /admin/users/:id/toggle-active`, `PATCH /admin/users/:id/role`,
  `DELETE /admin/users/:id`, `GET /admin/roles`.

## UI / components used

- shadcn-vue `form`, `input`, `input-otp`/`pin-input`, `button`, `card`,
  `dialog`/`alert-dialog`, `table`, `badge`, `dropdown-menu`, `tabs`, `switch`,
  `avatar`, `skeleton`, `sonner` (toasts); `pager` widget for the users table.

## Acceptance criteria

- [ ] Sign-up and sign-in work against the live API; access token held in memory, refresh persisted in localStorage; `loadSession` hydrates the bearer on startup.
- [ ] Sign-in returning `requires_two_fa` stashes `pending_token`, routes to `/two-factor`; verify completes login and persists tokens.
- [ ] Refresh rotation works: expired access → silent refresh; revoked refresh → forced sign-out and redirect to `/sign-in`.
- [ ] Router guard protects `requiresAuth` routes (unauthenticated → `/sign-in`); authenticated users land on the dashboard.
- [ ] Email verification (link token) and forgot → reset password flows work with field-level error surfacing from `ApiError.fields`.
- [ ] Full 2FA cycle in settings: status → setup (secret + QR) → enable (backup codes) → disable.
- [ ] Settings: profile edit (`PATCH /profile/me`), change password, sessions list + revoke / revoke-all render with loading/error/empty/success states.
- [ ] Admin `/users`: list (paginated), toggle-active, assign role, delete, roles list — gated to `ADMIN` via `use-permission` + guard.
- [ ] All auth copy + validation errors localized in en/ru/uz; assertions key on `error.code`, not message text.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
