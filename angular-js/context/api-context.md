# API Context — Backend Surface (consumed)

Machine-readable contract for the backend this client **consumes** (the
`express-monolith-api` modular monolith). This frontend does not define the API; it
binds to it. `context/api-rules.md` says how to call endpoints; this file IS the
contract they share. Keep it consistent with `context/architecture.md`.

## Base & versioning

- Base URL comes from `environments/*` (`apiBaseUrl`), e.g. `https://<host>/api/v1`.
  Never hardcode the host in components/services.
- All resource endpoints live under `/api/v1`.

## Response envelope (every endpoint, no exceptions)

Mirror these in `core/models` and unwrap them in the service layer.

```ts
// success
interface SuccessResponse<T> {
  success: true;
  data: T;
  meta?: PaginationMeta;          // present on paginated lists
}

// error
interface ErrorResponse {
  success: false;
  error: { code: string; message: string; fields?: Record<string, string[]> };
}

interface PaginationMeta { page: number; limit: number; total: number; totalPages: number }
```

The error is an **object** (`{ code, message, fields? }`), not a bare string. Branch
UI logic on `error.code`; `message` is server-localized and may change.

## Auth model (client side)

- Mechanism: **JWT** — access token (short-lived, ~15m) + refresh token (~7d) issued
  by the backend on login. Sent as `Authorization: Bearer <accessToken>`.
- **Interceptor-driven**: `auth.interceptor` attaches the bearer token to outgoing
  requests and, on `401`, attempts a single refresh-and-retry before forcing logout +
  redirect to login. Components/services never attach the header by hand.
- Token storage handled by `core/auth` (access token in memory where possible; refresh
  per the project's chosen strategy). Never log tokens.
- Route guards (`auth.guard`, `role.guard`) gate protected/admin routes client-side;
  the backend remains the source of truth (handle `401`/`403` from any call).
- Public surfaces: login/register, password reset, health — no token required.

## Pagination convention

- Request: `?page=1&limit=20` (`page` ≥ 1, default 1; `limit` 1–100, default 20).
  List calls add their own filters as extra query params.
- Response: items in `data`, `meta: { page, limit, total, totalPages }` per the
  envelope. Build list UIs (pager, infinite scroll) off `meta`.

## Validation convention

- The backend validates with Zod and returns `400` with field errors in
  `error.fields` (a `Record<string, string[]>`) plus a human-readable `error.message`.
- Client mirrors validation in reactive forms for UX, but always surfaces
  `error.fields` from the server response on submit failures.

## Errors / status conventions

- `400` validation (read `error.fields`), `401` unauthenticated (interceptor refresh),
  `403` forbidden (role/permission), `404` not found (or hidden-resource), `409`
  conflict, `429` rate-limited, `5xx` server. Map each to a typed client error in
  `error.interceptor` and a user-facing state.

## Consumed-endpoints registry (living document)

Updated by Builders with every backend endpoint this client calls; checked before
adding a new call (reuse the existing service method > duplicate); audited by /review.
One row per consumed `/api/v1` endpoint.

| Method | Path | Auth | Feature | Client service | Purpose |
| --- | --- | --- | --- | --- | --- |
| _(empty — first row added by the first feature that calls the backend)_ |

## Invariants

1. No backend call outside a `core`/feature service — components never call `HttpClient`.
2. No request/response untyped — every shape is defined in `core/models`.
3. No response handled outside the envelope (unwrap success / branch on `error.code`).
4. Every consumed endpoint appears in the registry above.
