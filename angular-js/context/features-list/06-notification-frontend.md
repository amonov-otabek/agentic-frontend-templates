# Feature 06 — Notifications (Frontend)

- Feature ID: FEAT-06
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01 (foundation), FEAT-02 (auth — these are authenticated screens)

> Source of truth for the frontend **notification** surface, consuming the backend
> `notification` domain. This project is **empty (no code exists yet)** — everything
> below is **TO BUILD**. `/parse-features 06` turns the `## Tasks` section into the
> per-feature task file.

## Goal

Build an authenticated **notifications** surface in the Angular SPA: a notifications
feed with an unread count, mark-read / mark-all-read actions, a header bell with a
live unread badge, and a per-channel preferences screen — driven by the backend
`notification` endpoints, guarded, typed, themed, and localized.

## Scope

### Routes / pages (`features/notification/`)

A lazy `notification.routes.ts` (under the protected group):

| Page | Route | Purpose |
| --- | --- | --- |
| Feed | `/notifications` | paginated feed; read/unread; mark-read + mark-all-read |
| Preferences | `/notifications/preferences` | per-channel / per-type opt-in toggles |

Plus a header **notification bell** component (in the layout shell) showing the unread
count and a recent-items dropdown.

### Components (`features/notification/components/`)

`notification-bell` (unread badge + dropdown) · `notification-feed` (paginated list) ·
`notification-item` (read/unread state, mark-read) · `mark-all-read-button` ·
`preferences-form` (channel/type toggles via Reactive Forms) · empty/error/loading
states.

### Services / state (`features/notification/services/`)

- `notification.service` wrapping the core `api.service`; typed `Notification`,
  `NotificationPreference` models in `features/notification/models/`.
- An unread-count signal kept in sync (polled or refreshed on focus; pluggable for a
  live transport later); feed data via RxJS bridged to signals; mark-read does an
  optimistic, immutable update with rollback on error.

### Guards

- `auth.guard` (authenticated; notifications are per-user).

## Consumed API

All under `/api/v1`, envelope `{ success, data, error, meta }` (lists carry `meta`).
Bind to the backend `notification` domain:

- `GET /notifications` — paginated feed (`?page&limit&unread`).
- `GET /notifications/unread-count` — current unread count for the badge.
- `PATCH /notifications/:id/read` — mark one read.
- `PATCH /notifications/read-all` — mark all read.
- `GET /notifications/preferences` — read per-channel/type preferences.
- `PUT /notifications/preferences` — update per-channel/type preferences.

Register every consumed endpoint in `context/api-context.md`.

## UI / components used

- Shared primitives (card, badge/chip, button, input/toggle) + the new notification
  components; the bell lives in the layout shell.
- Unread state via badge tokens; every async surface implements
  loading / empty / error / success; OnPush throughout; copy localized.

## Acceptance criteria

- [ ] `/notifications` lists a paginated feed driven by `meta`, distinguishing
      read/unread, with empty/error/loading states.
- [ ] Mark-one-read and mark-all-read work with optimistic immutable updates and
      rollback on error.
- [ ] Header bell shows the live unread count (badge) and a recent-items dropdown;
      count refreshes (poll/focus) and updates after read actions.
- [ ] `/notifications/preferences` reads and updates per-channel/type toggles via
      Reactive Forms; server `error.fields` surfaced on failure.
- [ ] Unread-count + feed state in signals; data via RxJS bridged to signals.
- [ ] All requests/responses typed against `features/notification/models`; components
      never call `HttpClient`; consumed endpoints registered in `api-context.md`.
- [ ] Routes guarded (authenticated); copy localized; badges from tokens.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
