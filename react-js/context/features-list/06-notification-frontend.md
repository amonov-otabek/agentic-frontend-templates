# Feature 06 — Notifications (Frontend)

- Feature ID: FEAT-06
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded dashboard surface)

> Source of truth for the authenticated **notifications** surface: a notification feed with
> unread tracking + mark-read actions, and a preferences form. Consumes the backend
> `notification` domain.

## Goal

Give authenticated users a notifications surface on `/notifications`: read the feed, see
the unread count, mark one or all read, and manage delivery preferences — all through
TanStack Query against the backend notification API.

## Scope

- **Notifications page** (`features/notification/pages/notifications-page`, route
  `/notifications` under `DashboardLayout`) — orchestrates the feed + preferences.
- **Notification list** (`components/notification-list`) — paginated feed with an
  unread-only filter; per-item Mark-read and a Mark-all-read action; visual read/unread state.
- **Unread count** — `useUnreadCount` drives a badge (suitable for the dashboard header /
  user nav) and refreshes after mark-read.
- **Preferences form** (`components/preferences-form`) — channel/category toggles; loads
  current preferences and persists via PUT (optimistic update where safe).
- **Queries** (`queries/notification.queries`) — `useNotifications`, `useUnreadCount`,
  `usePreferences`, `useUpdatePreferences`, `useMarkRead`, `useMarkAllRead`; mutations
  invalidate the feed + unread count. Typed via `types/notification.types`
  (`AppNotification`, `NotificationPreferences`, `NotificationListQuery`).
- Loading / error / empty / success states.
- Out of scope: real-time push/socket delivery (not wired in this template), per-notification deep links.

## Consumed API

Base `/api/v1/notifications`, envelope `{ success, data, meta? }`.

- `GET /notifications?page&limit&unread_only` → paginated `AppNotification[]` with `meta`.
- `GET /notifications/unread-count` → `{ unread }`.
- `GET /notifications/preferences` → `NotificationPreferences`.
- `PUT /notifications/preferences` → `NotificationPreferences`.
- `PATCH /notifications/:id/read` → `AppNotification`.
- `PATCH /notifications/read-all` → `{ updated }`.

## UI / components used

- `features/notification/components/{notification-list,preferences-form}`.
- shadcn `card`, `table`/`item`, `switch`, `checkbox`, `badge`, `button`, `tabs`,
  `skeleton`, `empty`, `sonner`; widgets `pager`; `shared/hooks/use-pagination`;
  `DashboardLayout`.

## Acceptance criteria

- [ ] `/notifications` renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] The feed paginates, supports an `unread_only` filter, and shows read/unread state.
- [ ] The unread-count badge reflects `GET /notifications/unread-count` and updates after
      mark-read / mark-all-read.
- [ ] Mark-read and mark-all-read call their endpoints and invalidate the feed + unread count.
- [ ] The preferences form loads current prefs and persists via `PUT`; success/error toast;
      optimistic update reverts on error.
- [ ] Loading, error, and empty states render.
- [ ] All HTTP flows through `api/notification.api` + `queries/notification.queries`;
      responses typed via `types/notification.types`; no `fetch` in components.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
