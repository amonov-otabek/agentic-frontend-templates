# Feature 06 — Notifications (Frontend)

- Feature ID: FEAT-06
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01, 02

> Source of truth for the **notification** feature slice — an authenticated
> notification feed with unread tracking and a preferences form, over the backend
> notification domain. `## Tasks` below is left for `/parse-features`.

## Goal

An authenticated notifications area: a paginated feed with read/unread state, an
unread count, mark-read / mark-all-read actions, and a preferences form — through
TanStack Query in the dashboard shell.

## Scope

- **Route:** `src/app/[locale]/(protected)/notifications/page.tsx` (thin) → renders
  `features/notification/pages/notifications-page.tsx`.
- **Feed** (`notification-list.tsx`): paginated `AppNotification` rows, an
  unread-only filter, per-row mark-read, and a mark-all-read action; unread count
  badge.
- **Preferences** (`preferences-form.tsx`): edit channel preferences
  (`NotificationPreferences`) and save.
- **Queries:** `queries/notification.queries.ts` — `useNotifications(query)`,
  `useUnreadCount`, `useMarkRead`, `useMarkAllRead`, `usePreferences`,
  `useUpdatePreferences`; mutations invalidate the feed + unread-count keys.
- **Types:** `types/notification.types.ts` — `AppNotification`,
  `NotificationListQuery`, `NotificationPreferences`.
- **Out of scope:** real-time push / websocket delivery (backend-owned); per-event
  templating.

## Consumed API

Base `${NEXT_PUBLIC_API_URL}/api/v1`, envelope; called only from
`features/notification/api/notification.api.ts`.

- `GET /notifications?page&limit&unread_only` → paginated `AppNotification[]`
- `GET /notifications/unread-count` → `{ unread: number }`
- `PATCH /notifications/:id/read` → `AppNotification`
- `PATCH /notifications/read-all` → `{ updated: number }`
- `GET /notifications/preferences` → `NotificationPreferences`
- `PUT /notifications/preferences` → `NotificationPreferences`

## UI / components used

- `features/notification/components/{notification-list,preferences-form}.tsx`.
- Shared UI: `card`, `badge` (unread), `switch`/`checkbox` (preferences), `button`,
  `table`/`item`, `pagination`/`pager`, `empty`, `skeleton`, `sonner`;
  `use-pagination`, `use-toast` hooks.

## Acceptance criteria

- [ ] Notifications route is guarded and renders in the dashboard shell.
- [ ] Feed paginates against `GET /notifications` with an unread-only filter.
- [ ] Unread count renders from `GET /notifications/unread-count`.
- [ ] Mark-read (`PATCH /notifications/:id/read`) and mark-all-read
      (`PATCH /notifications/read-all`) update state and invalidate the feed +
      unread-count keys.
- [ ] Preferences load (`GET …/preferences`) and save (`PUT …/preferences`) with a
      success toast; field errors mapped from `error.fields`.
- [ ] Loading / empty / error states handled; errors branch on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
