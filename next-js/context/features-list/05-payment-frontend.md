# Feature 05 — Payments & Invoices (Frontend)

- Feature ID: FEAT-05
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01, 02

> Source of truth for the **payment** feature slice — an authenticated payments +
> invoices surface over the backend payment domain: list/create/refund payments and
> list/pay invoices. `## Tasks` below is left for `/parse-features`.

## Goal

An authenticated payments area: browse payments (filter by status), create a new
payment, refund an existing one, and browse + pay invoices — fetched and mutated
through TanStack Query in the dashboard shell.

## Scope

- **Route:** `src/app/[locale]/(protected)/payments/page.tsx` (thin) → renders
  `features/payment/pages/PaymentsPage.tsx`.
- **Payments list** (`PaymentList.tsx`): paginated `Payment` rows with a status
  filter and a per-row refund action.
- **Create payment** (`PaymentCreateForm.tsx`): Zod-validated form (amount /
  currency / metadata) calling `createPayment`.
- **Invoices list** (`InvoiceList.tsx`): paginated `Invoice` rows with a status
  filter and a per-row pay action.
- **Queries:** `queries/payment.queries.ts` — `usePayments(query)`,
  `useCreatePayment`, `useRefundPayment`, `useInvoices(query)`, `usePayInvoice`;
  mutations invalidate the respective list keys.
- **Validators / types:** `validators/payment.validator.ts`;
  `types/payment.types.ts` — `Payment`, `Invoice`, `CreatePaymentDto`,
  `PaymentListQuery`, `InvoiceListQuery`.
- **Out of scope:** real payment-gateway integration / checkout redirects
  (backend-owned); subscription management.

## Consumed API

Base `${NEXT_PUBLIC_API_URL}/api/v1`, envelope; called only from
`features/payment/api/payment.api.ts`.

- `GET /payments?page&limit&status` → paginated `Payment[]`
- `POST /payments` → `Payment`
- `POST /payments/:id/refund` → `Payment`
- `GET /invoices?page&limit&status` → paginated `Invoice[]`
- `PATCH /invoices/:id/pay` → `Invoice`

## UI / components used

- `features/payment/components/{payment-list,payment-create-form,invoice-list}.tsx`.
- Shared UI: `table`, `badge` (status), `select`/`native-select` (filter), `dialog`
  (create/refund/pay confirm), `field`/`input`/`label`, `button`,
  `pagination`/`pager`, `empty`, `skeleton`, `sonner`; `usePagination`,
  `use-toast` hooks.

## Acceptance criteria

- [ ] Payments route is guarded and renders in the dashboard shell.
- [ ] Payments list paginates against `GET /payments` and filters by status.
- [ ] Create payment validates with Zod (field errors mapped from `error.fields`)
      and invalidates the payments list on success.
- [ ] Refund (`POST /payments/:id/refund`) updates the row; pay invoice
      (`PATCH /invoices/:id/pay`) updates the invoice row.
- [ ] Invoices list paginates against `GET /invoices` and filters by status.
- [ ] Loading / empty / error / success states handled; errors branch on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
