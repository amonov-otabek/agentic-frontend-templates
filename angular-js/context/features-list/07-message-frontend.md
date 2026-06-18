# Feature 07 — Messages (Frontend)

- Feature ID: FEAT-07
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01 (foundation), FEAT-02 (auth — these are authenticated screens)

> Source of truth for the frontend **message** surface, consuming the backend
> `message` domain (async email/SMS delivery). This project is **empty (no code
> exists yet)** — everything below is **TO BUILD**. `/parse-features 07` turns the
> `## Tasks` section into the per-feature task file.

## Goal

Build an authenticated **messages** surface in the Angular SPA: a compose form that
submits a message over a chosen channel (email / SMS) for async delivery, and a
paginated sent-messages list with per-message delivery status — driven by the backend
`message` endpoints, guarded, typed, themed, and localized.

## Scope

### Routes / pages (`features/message/`)

A lazy `message.routes.ts` (under the protected group):

| Page | Route | Purpose |
| --- | --- | --- |
| Messages | `/messages` | paginated sent-messages list with delivery status + channel filter |
| Compose | `/messages/compose` | compose + send a message (channel: email or SMS) |

(Compose may be a route or a modal launched from `/messages` — pick one and keep it
consistent.)

### Components (`features/message/components/`)

`compose-form` (recipient, channel select, subject [email], body — Reactive Forms +
typed validators) · `messages-table` (paginated, sortable) · `message-status-badge`
(queued / sent / delivered / failed) · `message-filter-bar` (channel, status, date) ·
`message-detail` (payload + status timeline) · empty/error/loading states.

### Services / state (`features/message/services/`)

- `message.service` wrapping the core `api.service`; typed `Message`, `MessageChannel`,
  `MessageStatus`, and DTO/query models in `features/message/models/`.
- View state via signals (filters, page, selection); list data via RxJS bridged to
  signals; after a successful submit, immutably refresh the list (status updates async
  on the backend — poll/refetch to reflect delivery).

### Guards

- `auth.guard` (authenticated; messages are per-user).

## Consumed API

All under `/api/v1`, envelope `{ success, data, error, meta }` (lists carry `meta`).
Bind to the backend `message` domain (async: submit enqueues, status updates later):

- `POST /messages` — submit a message; body includes `channel` (`email` | `sms`),
  recipient, and content; returns the created message with an initial status.
- `GET /messages` — paginated sent-messages (`?page&limit&channel&status&from&to`).
- `GET /messages/:id` — single message detail + delivery status.

Register every consumed endpoint in `context/api-context.md`.

## UI / components used

- Shared primitives (card, input, badge/chip, button, select) + the new message
  components.
- Delivery status via badge tokens; every async surface implements
  loading / empty / error / success; OnPush throughout; copy localized.

## Acceptance criteria

- [ ] Compose form (recipient, channel select email/SMS, subject for email, body)
      validates via Reactive Forms + typed validators and surfaces `error.fields`.
- [ ] Submitting calls `POST /messages`; success shows a confirmation and the message
      appears in the list with its initial status.
- [ ] `/messages` lists paginated messages driven by `meta`; channel/status/date
      filters update the query and refetch; status badges reflect delivery state.
- [ ] Message detail shows payload + delivery status timeline; list refetch/poll
      reflects async status changes.
- [ ] Filter/page/selection state in signals; data via RxJS bridged to signals;
      immutable updates.
- [ ] All requests/responses typed against `features/message/models`; components never
      call `HttpClient`; consumed endpoints registered in `api-context.md`.
- [ ] Routes guarded (authenticated); copy localized; badges from tokens.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
