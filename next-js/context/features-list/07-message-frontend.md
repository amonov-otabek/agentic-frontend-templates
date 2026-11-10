# Feature 07 — Messages (Frontend)

- Feature ID: FEAT-07
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01, 02

> Source of truth for the **message** feature slice — an authenticated compose +
> list surface over the backend message domain (async email/SMS delivery). `##
> Tasks` below is left for `/parse-features`.

## Goal

An authenticated messages area: compose a message (channel email/SMS), submit it
for async delivery, and browse the delivery list with status — through TanStack
Query in the dashboard shell.

## Scope

- **Route:** `src/app/[locale]/(protected)/messages/page.tsx` (thin) → renders
  `features/message/pages/MessagesPage.tsx`.
- **Compose** (`MessageComposeForm.tsx`): Zod-validated form — channel
  (`email` | `sms`), recipient, subject/body — calling `createMessage`.
- **List** (`MessageList.tsx`): paginated `Message` rows with a status filter
  (queued / sent / failed) and per-row detail.
- **Queries:** `queries/message.queries.ts` — `useMessages(query)`,
  `useMessage(id)`, `useCreateMessage`; create invalidates the messages list key.
- **Validators / types:** `validators/message.validator.ts`;
  `types/message.types.ts` — `Message`, `CreateMessageDto`, `MessageListQuery`.
- **Out of scope:** worker/provider delivery internals (backend-owned); inbound
  message threads / conversations.

## Consumed API

Base `${NEXT_PUBLIC_API_URL}/api/v1`, envelope; called only from
`features/message/api/message.api.ts`.

- `GET /messages?page&limit&status` → paginated `Message[]`
- `GET /messages/:id` → `Message`
- `POST /messages` → `Message` (queued for async delivery)

## UI / components used

- `features/message/components/{message-compose-form,message-list}.tsx`.
- Shared UI: `field`/`input`/`textarea`/`label`, `select`/`native-select`
  (channel + status filter), `button`, `table`, `badge` (status), `dialog`/`sheet`
  (detail), `pagination`/`pager`, `empty`, `skeleton`, `sonner`; `usePagination`,
  `use-toast` hooks.

## Acceptance criteria

- [ ] Messages route is guarded and renders in the dashboard shell.
- [ ] Compose validates with Zod (channel email/SMS), posts `POST /messages`, shows a
      success toast, and invalidates the list; field errors mapped from `error.fields`.
- [ ] List paginates against `GET /messages` and filters by status; per-row detail
      via `GET /messages/:id`.
- [ ] Loading / empty / error / success states handled; errors branch on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
