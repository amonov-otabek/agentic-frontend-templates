# API Context — Backend Surface (consumed)

Machine-readable contract this client **depends on**. The backend monolith owns and
builds these endpoints; this file is the shape the client trusts and the living registry of
what we actually call. The typed client (`shared/libs/api.ts`) encodes this contract — keep
them consistent with `context/architecture.md`.

## Base URL

- Configured via `NEXT_PUBLIC_API_URL` (default `http://localhost:3000`), trailing slash stripped.
- All endpoints live under `/api/v1` (e.g. `/api/v1/auth/sign-in`).

## Response envelope (every endpoint, no exceptions)

Encoded once in `shared/libs/api.ts`; the client unwraps `data` for callers and throws on failure.

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

interface ApiMeta { page: number; limit: number; total: number; totalPages: number }
```

The error is an **object** (`{ code, message, fields? }`), not a bare string. On failure the
client raises `ApiError(status, code, message, fields)` — branch on `error.code`, never on
message text (messages are i18n-localized server-side). `204 No Content` resolves to `data: undefined`.

## Auth model

- Mechanism: **JWT** — access token + refresh token pair returned on sign-in / sign-up.
  The client holds the access token in memory (`setAccessToken`) and persists both to
  `localStorage`; the refresh token is used to re-auth. `Authorization: Bearer <token>` is
  attached automatically to every request; `credentials: 'include'` is sent.
- 2FA: when sign-in returns `requires_two_fa`, the `pending_token` is stashed in
  `sessionStorage` and exchanged at `/api/v1/2fa/verify` for the real token pair.
- Route guarding is client-side: the `(protected)` layout redirects unauthenticated users to
  the `(auth)` group. Public endpoints: sign-in/up, forgot/reset password, verify-email links.

## Pagination convention

- Request: `?page=1&limit=20`. `limit` is 1–100 (mirrors `shared/libs/validators.ts`
  `paginationQuerySchema`, default page 1 / limit 20).
- Response: `meta: { page, limit, total, totalPages }`. Use `api.getPaginated<T>(path)` → `{ data, meta }`.

## Validation convention

- The backend returns field errors in `error.fields` (`Record<string, string[]>`) on `400`.
  The client mirrors that shape with Zod via `formatZodErrors`, so client- and server-side
  validation errors map onto form fields identically.

## Consumed endpoints (living registry)

Updated by Builders with every backend endpoint the client calls (in a feature's `api/`
folder); checked before adding a new call (reuse > duplicate); audited by `/review`.
One row per consumed `/api/v1` endpoint. Auth = whether a bearer token is required.

| Method | Path | Auth | Feature | Called from | Purpose |
| --- | --- | --- | --- | --- | --- |
| POST | `/api/v1/auth/sign-in` | no | identity | `identity/api/auth.api.ts` | email+password sign-in (may return 2FA challenge) |
| POST | `/api/v1/auth/sign-up` | no | identity | `identity/api/auth.api.ts` | register account |
| POST | `/api/v1/auth/refresh` | no | identity | `identity/api/auth.api.ts` | exchange refresh token for a new pair |
| POST | `/api/v1/auth/logout` | yes | identity | `identity/api/auth.api.ts` | revoke refresh token / sign out |
| POST | `/api/v1/auth/forgot-password` | no | identity | `identity/api/auth.api.ts` | request password reset email |
| POST | `/api/v1/auth/reset-password` | no | identity | `identity/api/auth.api.ts` | reset password with token |
| POST | `/api/v1/auth/verify-email-link` | no | identity | `identity/api/auth.api.ts` | confirm email via token |
| POST | `/api/v1/2fa/verify` | no | identity | `identity/api/auth.api.ts` | verify 2FA code, return token pair |
| GET | `/api/v1/profile/me` | yes | identity | `identity/api/auth.api.ts` | current authenticated user |

_(Additional rows added per feature: profile, sessions, twofa, admin/users, messages,
notifications, payments, audit, analytics.)_

## Invariants

1. No backend call outside a feature's `api/` folder, and every call goes through the typed
   client — no bare `fetch`.
2. No response shape trusted outside the envelope; always unwrap via the client.
3. DTOs/response types are defined once in the feature's `types/` and imported — never
   hand-duplicated across layers.
4. Every consumed endpoint appears in this registry.
