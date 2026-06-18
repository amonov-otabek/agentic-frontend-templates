# Feature 06 — Notifications (Frontend)

- Feature ID: FEAT-06
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth + guarded dashboard)

> Source of truth for the authenticated **notifications** surface: feed, unread
> badge, and preferences. `/parse-features 06` turns the `## Tasks` section into
> `context/tasks-list/06-notification-frontend-tasks.md`.

## Goal

Let authenticated users read their notification feed, track an unread count,
mark items read (single + all), and manage delivery preferences — served by the
backend notification domain, themed and localized.

## Scope (real pages, routes, components, composables)

- **Page** (`src/features/notification/pages/notifications-page.vue`): mounted at
  `/notifications` under `DashboardLayout` (`requiresAuth`,
  `meta.titleKey:'nav.notifications'`).
- **Components** (`src/features/notification/components/`): `notification-list.vue`
  (paginated feed, unread-only toggle, mark-read / mark-all-read),
  `preferences-form.vue` (per-channel preferences).
- **Queries** (`queries/notification.queries.ts`): `useNotifications(query)`,
  `useUnreadCount()`, `usePreferences()`, `useMarkRead()`, `useMarkAllRead()`,
  `useUpdatePreferences()`.
- **Types** (`types/notification.types.ts`): `AppNotification`,
  `NotificationPreferences`, `NotificationListQuery`.
- **Widget**: unread-count badge surfaced in the dashboard header/user-nav.
- Out of scope: real-time push/websocket delivery (feed is fetch + refetch-on-focus).

## Consumed API (`/api/v1/notifications`, shared envelope)

- `GET /notifications?page&limit&unread_only` → `Paginated<AppNotification>`.
- `GET /notifications/unread-count` → `{ unread }`.
- `GET /notifications/preferences` → `NotificationPreferences`.
- `PUT /notifications/preferences` → `NotificationPreferences`.
- `PATCH /notifications/:id/read` → `AppNotification`.
- `PATCH /notifications/read-all` → `{ updated }`.

## UI / components used

- shadcn-vue `card`/`item`, `switch`/`checkbox` (preferences), `badge` (unread),
  `button`, `skeleton`, `empty`, `pager`/`use-pagination`, `sonner`.

## Acceptance criteria

- [ ] `/notifications` is guarded and reachable from the dashboard.
- [ ] Feed paginates with an unread-only toggle; mark-read and mark-all-read update the cache optimistically where safe.
- [ ] Unread-count badge renders in the header and refreshes after read actions.
- [ ] Preferences form loads and persists via `PUT /notifications/preferences` with success/error states.
- [ ] Loading/error/empty states throughout; labels localized in en/ru/uz; errors keyed on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
