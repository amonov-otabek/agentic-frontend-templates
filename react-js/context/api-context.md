# API Context — Consumed Backend Surface

Machine-readable contract for the **backend this SPA consumes** (`express-monolith-api`,
`/api/v1`). The client does not define this contract — it mirrors it. This file is the
shared reference for every feature `api/` and `queries/` module, and the living registry
of endpoints the client actually calls. Keep it consistent with `context/architecture.md`
and `shared/libs/api.ts`.

## Base URL & client

- Origin comes from `VITE_API_URL` (default `http://localhost:3000`); the `/api/v1`
  prefix is added by feature `api/` paths.
- All requests go through `shared/libs/api.ts` (`api.get/getPaginated/post/put/patch/delete`),
  which attaches auth, unwraps the envelope, and throws a typed `ApiError` on failure.
  Requests send `credentials: "include"`.

## Response envelope (every endpoint, no exceptions)

Mirrored in `shared/libs/api.ts`. The client unwraps `data` (and `meta` for lists);
components never see the raw envelope.

```ts
// success
interface ApiSuccess<T> {
  success: true;
  data: T;
  meta?: ApiMeta;            // present on paginated lists
}

// error
interface ApiFailure {
  success: false;
  error: { code: string; message: string; fields?: Record<string, string[]> };
}

interface ApiMeta { total: number; page: number; limit: number }
```

On a non-2xx status, a `false` envelope, or a parse failure, the client throws
`ApiError(status, code, message, fields?)`. Branch on `error.code` / `error.fields` —
**never** on `message` (messages are i18n-localized server-side). A `204` resolves to
`data: undefined`.

## Auth model

- Mechanism: **JWT bearer**. Sign-in returns an access + refresh token pair; the access
  token is held in memory (`setAccessToken`) and sent as `Authorization: Bearer <token>`
  on every request. Tokens are persisted in `localStorage` and rehydrated on startup
  (`features/identity/services/auth.service.ts`, `loadSession()`).
- **2FA**: if sign-in returns `requires_two_fa`, the client stashes a short-lived
  pending token (sessionStorage) and completes auth at `/two-factor`.
- **Refresh**: the refresh token (also persisted) is used to obtain new access tokens
  and to invalidate the session on sign-out (`logout`).
- **Guards are UX only**: route guards check for a stored session and redirect to
  `/sign-in`. The backend enforces real authorization; a 401/403 from the API is the
  source of truth and should clear the session.

## Pagination convention

- Request: `?page=1&limit=20` query params (sent via `api.getPaginated`, `options.params`).
- Response: items in `data`, counts in `meta: { total, page, limit }`. Pages compute
  `totalPages = ceil(total / limit)`; see `shared/hooks/use-pagination.ts`.

## Validation convention

- Library: **Zod** (matching the backend). One schema per form/input surface in the
  feature's `validators/`. Server-side field errors come back in `error.fields`
  (`Record<string, string[]>`) on a 400 and map directly onto form fields.

## Errors / events

- The client is a consumer: it does not publish events. Domain side effects (welcome
  email, audit, analytics) happen server-side off the same requests the client makes.
- Real-time push (if used) would reuse the same JWT on a Socket.IO handshake — not wired
  by default in this template.

## Consumed-endpoints registry (living document)

Updated by Builders with every backend endpoint a feature calls; checked before adding a
new call (reuse the existing `api/` function > duplicate); audited by `/review`. One row
per distinct `/api/v1` endpoint the client consumes.

| Method | Path | Auth | Feature | Client caller |
| --- | --- | --- | --- | --- |
| _(empty — first row added by the first feature wired to the API)_ |

## Invariants

1. No `fetch` outside `shared/libs/api.ts`; no `/api/v1` call outside a feature `api/`
   module listed in this registry.
2. No component reads the raw envelope — only unwrapped `data` / `{ data, meta }`.
3. Request/response types are defined once per feature in `types/` and imported —
   never duplicated inline.
4. Auth failures (401/403) clear the session and route to `/sign-in`; never retry blindly.
