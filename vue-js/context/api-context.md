# API Context — Consumed Backend Surface

Machine-readable contract for the backend this SPA **consumes** (the
`express-monolith-api`): the envelope, auth model, shared conventions, and the living
registry of endpoints the client calls. `context/api-rules.md` says how to wire a
feature's data layer; this file IS the contract that layer depends on. Keep it
consistent with `context/architecture.md` and the backend's own API docs.

## Response envelope (every endpoint, no exceptions)

The client mirrors the backend envelope in `shared/libs/api.ts`. The `api` helpers
unwrap `data` on success and throw a typed `ApiError` on failure — feature code works
with plain data, never the raw envelope.

```ts
// success
interface ApiSuccess<T> {
  success: true;
  data: T;
  meta?: ApiMeta;                 // present on paginated lists
}

// error
interface ApiFailure {
  success: false;
  error: { code: string; message: string; fields?: Record<string, string[]> };
}

interface ApiMeta { total: number; page: number; limit: number }

// thrown by the client on !ok / success:false
class ApiError extends Error { status: number; code: string; fields?: Record<string, string[]> }
```

Note the error is an **object** (`{ code, message, fields? }`), not a bare string.
Assert/branch on `error.code` (stable), not `message` (i18n-localized).

## Auth model

- Mechanism: **JWT bearer**. Sign-in returns an access + refresh token pair; the auth
  service (`features/identity/services/auth.service.ts`) persists both to localStorage
  and seeds the in-memory bearer via `setAccessToken`. The api client attaches
  `Authorization: Bearer <token>` to every request automatically.
- Storage: `auth.access_token` / `auth.refresh_token` in localStorage; rehydrated on
  startup (`loadSession`). A short-lived 2FA challenge token lives in sessionStorage
  (`auth.pending_2fa`) across the sign-in → two-factor navigation.
- Guarding: `requiresAuth` routes are gated by a `vue-router` `beforeEach` guard
  (`isAuthenticated()` checks the stored access token); no session → redirect to
  `/sign-in`. Public: home, sign-in/up, forgot/reset password, verify-email.
- `credentials: "include"` is set on requests (cookie support if the backend uses it).

## Pagination convention

- Request: list endpoints take `?page=1&limit=20` (validated client-side by
  `paginationQuerySchema`: `page` ≥ 1 default 1, `limit` 1–100 default 20).
- Read paginated lists via `api.getPaginated<T>(path, { params })` → returns
  `{ data: T[], meta: { total, page, limit } }`. Use `use-pagination` for UI state.

## Validation convention

- Library: **Zod** (matches the backend). One schema per form/input surface in the
  feature's `validators/`; shared schemas in `shared/libs/validators`. Forms bind via
  `@vee-validate/zod`.
- On a 400, the backend returns field errors in `error.fields`; map them onto the form
  via the thrown `ApiError.fields`. `formatZodErrors` produces the same shape locally
  for client-side failures.

## Base URL & paths

- Origin from `import.meta.env.VITE_API_URL` (default `http://localhost:3000`),
  trailing slash stripped. Feature `api/*.ts` functions add the `/api/v1/...` path.
- The 204 No Content case returns `{ success: true, data: undefined }` from the client.

## Endpoint registry (living document)

Updated by Builders with every endpoint a feature calls; checked before adding new
calls (reuse > duplicate); audited by `/review`. One row per consumed `/api/v1`
endpoint. (Backend ops/health endpoints are not consumed by the client and are exempt.)

| Method | Path | Auth | Feature | Purpose |
| --- | --- | --- | --- | --- |
| _(empty — first row added by the first feature's data layer)_ |

## Invariants

1. No backend path named outside a feature `api/*.ts` function.
2. No component reads the raw envelope — work with the unwrapped data the client returns.
3. DTOs/view models for a feature are defined once in `features/<name>/types` and
   imported — never duplicated by hand.
4. All server reads/writes go through TanStack Query hooks in `queries/*.queries.ts`.
