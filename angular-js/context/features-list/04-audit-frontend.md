# Feature 04 — Audit Log Viewer (Frontend)

- Feature ID: FEAT-04
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01 (foundation), FEAT-02 (auth — these are guarded, admin screens)

> Source of truth for the frontend **audit** surface, consuming the backend `audit`
> domain. This project is **empty (no code exists yet)** — everything below is
> **TO BUILD**. `/parse-features 04` turns the `## Tasks` section into the per-feature
> task file.

## Goal

Build an authenticated **audit log viewer** with a security **alerts** panel in the
Angular SPA: a searchable, filterable, paginated audit-event table, an event detail
view, and an alerts list with a resolve action — driven by the backend `audit`
endpoints, guarded (admin), typed, themed, and localized.

## Scope

### Routes / pages (`features/audit/`)

A lazy `audit.routes.ts` (under the protected/admin group):

| Page | Route | Purpose |
| --- | --- | --- |
| Audit log | `/audit` | searchable + filterable paginated event table; row → detail |
| Alerts | `/audit/alerts` | security alerts list with resolve action |

### Components (`features/audit/components/`)

`audit-table` (paginated, sortable; columns: time, actor, action, target, ip/agent) ·
`audit-filter-bar` (date range, actor, action, free-text search) · `audit-detail`
(full event payload, read-only) · `alerts-list` · `alert-row` (severity badge +
resolve button) · empty/error/loading states.

### Services / state (`features/audit/services/`)

- `audit.service` wrapping the core `api.service`; typed `AuditEvent`, `AuditAlert`,
  and query-param models in `features/audit/models/`.
- View state via signals (filters, page, selected event); list data via RxJS bridged
  to signals; resolve action triggers an immutable refresh of the alerts list.

### Guards

- `auth.guard` + `role.guard` (`admin`) — audit data is admin-scoped.

## Consumed API

All under `/api/v1`, envelope `{ success, data, error, meta }` (lists carry `meta`).
Bind to the backend `audit` domain:

- `GET /audit` — paginated audit events (`?page&limit` + `from&to&actor&action&q`).
- `GET /audit/:id` — single event detail (if exposed; else detail from the row).
- `GET /audit/alerts` — paginated security alerts.
- `PATCH /audit/alerts/:id/resolve` — mark an alert resolved.

Register every consumed endpoint in `context/api-context.md`.

## UI / components used

- Shared primitives (card, input, badge/chip, button) + the new audit components.
- Severity rendered via badge tokens; every async surface implements
  loading / empty / error / success; OnPush throughout; timestamps localized.

## Acceptance criteria

- [ ] `/audit` lists paginated events driven by `meta`; search + date/actor/action
      filters update the query and refetch.
- [ ] Row selection opens a read-only event detail with the full payload.
- [ ] `/audit/alerts` lists alerts with severity badges; resolve calls
      `PATCH /audit/alerts/:id/resolve` and refreshes the list immutably.
- [ ] Filter/page/selection state in signals; data via RxJS bridged to signals.
- [ ] All requests/responses typed against `features/audit/models`; components never
      call `HttpClient`; consumed endpoints registered in `api-context.md`.
- [ ] Routes guarded by `auth.guard` + `role.guard` (admin only; `403` handled).
- [ ] Severity colors and all visuals from design tokens; timestamps localized.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
