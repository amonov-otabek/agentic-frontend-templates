# Feature 07 — Messages (Frontend)

- Feature ID: FEAT-07
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth + guarded dashboard)

> Source of truth for the authenticated **messages** surface: compose + list.
> `/parse-features 07` turns the `## Tasks` section into
> `context/tasks-list/07-message-frontend-tasks.md`.

## Goal

Let authenticated users compose async messages (email / sms channel) and browse
the sent list with delivery status — served by the backend message domain
(queued delivery), themed and localized.

## Scope (real pages, routes, components, composables)

- **Page** (`src/features/message/pages/messages-page.vue`): mounted at
  `/messages` under `DashboardLayout` (`requiresAuth`, `meta.titleKey:'nav.messages'`).
- **Components** (`src/features/message/components/`): `message-compose-form.vue`
  (Zod-validated; channel `email`|`sms`, recipient, body), `message-list.vue`
  (paginated, status filter, per-row delivery status).
- **Queries** (`queries/message.queries.ts`): `useMessages(query)`,
  `useMessage(id)`, `useCreateMessage()`.
- **Types** (`types/message.types.ts`): `Message`, `CreateMessageDto`,
  `MessageListQuery`.
- Out of scope: threaded conversations / replies (the backend is one-shot
  async delivery) — flag if scoped.

## Consumed API (`/api/v1/messages`, shared envelope)

- `GET /messages?page&limit&status` → `Paginated<Message>`.
- `GET /messages/:id` → `Message`.
- `POST /messages` → `Message` (create / enqueue).

## UI / components used

- shadcn-vue `form`/`input`/`textarea`/`select` (channel), `button`, `table`,
  `badge` (status), `skeleton`, `empty`, `pager`/`use-pagination`, `sonner`.

## Acceptance criteria

- [ ] `/messages` is guarded and reachable from the dashboard.
- [ ] Compose form is Zod-validated (channel/recipient/body); submit calls `POST /messages` and refetches the list on success.
- [ ] Message list paginates and filters by status; per-row delivery status renders as a badge.
- [ ] Loading/error/empty/success states throughout; labels localized in en/ru/uz; errors keyed on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
