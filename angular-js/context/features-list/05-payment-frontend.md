# Feature 05 — Payments & Invoices (Frontend)

- Feature ID: FEAT-05
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01 (foundation), FEAT-02 (auth — these are authenticated screens)

> Source of truth for the frontend **payment** surface, consuming the backend
> `payment` domain. This project is **empty (no code exists yet)** — everything below
> is **TO BUILD**. `/parse-features 05` turns the `## Tasks` section into the
> per-feature task file.

## Goal

Build an authenticated **payments** surface in the Angular SPA: a paginated payments
list with status filters, a payment detail view, and an invoices list/detail with a
download action — driven by the backend `payment` endpoints, guarded, typed, themed,
and localized.

## Scope

### Routes / pages (`features/payment/`)

A lazy `payment.routes.ts` (under the protected group):

| Page | Route | Purpose |
| --- | --- | --- |
| Payments | `/payments` | paginated payments list with status + date filters |
| Payment detail | `/payments/:id` | single payment: amount, status timeline, method, refs |
| Invoices | `/payments/invoices` | paginated invoices list |
| Invoice detail | `/payments/invoices/:id` | invoice line items + download |

### Components (`features/payment/components/`)

`payments-table` (paginated, sortable) · `payment-status-badge` · `payment-filter-bar`
(status, date range) · `payment-detail-card` (amount, currency, status, timeline) ·
`invoices-table` · `invoice-detail` (line items + totals) · `invoice-download-button` ·
empty/error/loading states. Money/amount rendering uses a shared currency pipe;
currency is generic (driven by the API value, not hardcoded).

### Services / state (`features/payment/services/`)

- `payment.service` wrapping the core `api.service`; typed `Payment`, `Invoice`,
  `PaymentStatus`, and query-param models in `features/payment/models/`.
- View state via signals (filters, page, selection); list/detail data via RxJS bridged
  to signals; immutable updates.

### Guards

- `auth.guard` (authenticated). Apply `role.guard` only if a project scopes payments
  to admins — default is the owning user's own payments.

## Consumed API

All under `/api/v1`, envelope `{ success, data, error, meta }` (lists carry `meta`).
Bind to the backend `payment` domain:

- `GET /payments` — paginated payments (`?page&limit&status&from&to`).
- `GET /payments/:id` — single payment detail.
- `GET /invoices` — paginated invoices (`?page&limit&status`).
- `GET /invoices/:id` — invoice detail (line items, totals).
- `GET /invoices/:id/pdf` — rendered invoice PDF (PDFKit) for download.

Register every consumed endpoint in `context/api-context.md`.

## UI / components used

- Shared primitives (card, badge/chip, button, input) + the new payment components.
- Status via badge tokens; amounts via a currency pipe (locale + currency from data);
  every async surface implements loading / empty / error / success; OnPush throughout.

## Acceptance criteria

- [ ] `/payments` lists paginated payments driven by `meta`; status + date filters
      update the query and refetch.
- [ ] `/payments/:id` shows amount, currency, status timeline, method, and references.
- [ ] `/payments/invoices` lists invoices; `/payments/invoices/:id` shows line items +
      totals; download fetches the document and triggers a browser download.
- [ ] Amounts formatted via a currency pipe using the value's currency (no hardcoded
      currency symbol); statuses rendered via badge tokens.
- [ ] Filter/page/selection state in signals; data via RxJS bridged to signals;
      immutable updates.
- [ ] All requests/responses typed against `features/payment/models`; components never
      call `HttpClient`; consumed endpoints registered in `api-context.md`.
- [ ] Routes guarded (authenticated; admin-scoped only if the project requires it).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
