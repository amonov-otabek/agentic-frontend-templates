# Feature 05 — Payments & Invoices (Frontend)

- Feature ID: FEAT-05
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth + guarded dashboard)

> Source of truth for the authenticated **payments + invoices** surface.
> `/parse-features 05` turns the `## Tasks` section into
> `context/tasks-list/05-payment-frontend-tasks.md`.

## Goal

Let authenticated users view and act on payments and invoices: a paginated
payments list with create + refund, and a paginated invoices list with a pay
action, served by the backend payment domain, themed and localized.

## Scope (real pages, routes, components, composables)

- **Page** (`src/features/payment/pages/payments-page.vue`): mounted at
  `/payments` under `DashboardLayout` (`requiresAuth`, `meta.titleKey:'nav.payments'`).
- **Components** (`src/features/payment/components/`): `payment-list.vue`
  (paginated, status filter), `payment-create-form.vue` (Zod-validated create),
  `invoice-list.vue` (paginated, status filter, pay action).
- **Queries** (`queries/payment.queries.ts`): `usePayments(query)`,
  `useInvoices(query)`, `useCreatePayment()`, `useRefundPayment()`, `usePayInvoice()`.
- **Types** (`types/payment.types.ts`): `Payment`, `Invoice`, `CreatePaymentDto`,
  `PaymentListQuery`, `InvoiceListQuery`.
- Out of scope: payment-provider redirect/checkout integration and receipts (flag
  if scoped).

## Consumed API (`/api/v1`, shared envelope)

- `GET /payments?page&limit&status` → `Paginated<Payment>`.
- `POST /payments` → `Payment` (create).
- `POST /payments/:id/refund` → `Payment` (amount).
- `GET /invoices?page&limit&status` → `Paginated<Invoice>`.
- `PATCH /invoices/:id/pay` → `Invoice`.

## UI / components used

- shadcn-vue `table`, `form`/`input`/`number-field`, `button`,
  `dialog`/`alert-dialog` (refund/pay confirm), `badge` (status), `select`
  (filter), `skeleton`, `empty`, `pager`/`use-pagination`, `sonner`.

## Acceptance criteria

- [ ] `/payments` is guarded and reachable from the dashboard.
- [ ] Payments list paginates and filters by status; create form is Zod-validated and refetches on success.
- [ ] Refund action (with confirm) calls `POST /payments/:id/refund` and updates the row.
- [ ] Invoices list paginates and filters by status; pay action calls `PATCH /invoices/:id/pay` with confirm.
- [ ] Loading/error/empty/success states throughout; labels localized in en/ru/uz; errors keyed on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
