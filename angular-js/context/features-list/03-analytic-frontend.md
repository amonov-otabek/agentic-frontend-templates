# Feature 03 — Analytics Dashboard (Frontend)

- Feature ID: FEAT-03
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: FEAT-01 (foundation), FEAT-02 (auth — these are authenticated screens)

> Source of truth for the frontend **analytics** surface, consuming the backend
> `analytic` domain. This project is **empty (no code exists yet)** — everything below
> is **TO BUILD**. `/parse-features 03` turns the `## Tasks` section into the
> per-feature task file.

## Goal

Build an authenticated **analytics dashboard** in the Angular SPA: summary metric
cards, time-series and breakdown charts, and a reports view with date-range and
filter controls — all driven by the backend `analytic` endpoints, guarded, typed,
themed, and localized.

## Scope

### Routes / pages (`features/analytic/`)

A lazy `analytic.routes.ts` (under the protected group):

| Page | Route | Purpose |
| --- | --- | --- |
| Dashboard | `/analytics` | KPI summary cards + headline charts (overview) |
| Reports | `/analytics/reports` | filterable, date-ranged report tables + export-ready views |

### Components (`features/analytic/components/`)

`metric-card` (KPI tile with delta + trend) · `time-series-chart` ·
`breakdown-chart` (bar/pie) · `date-range-picker` · `filter-bar` · `report-table`
(sortable, paginated) · `chart-empty/error/loading` states. Charts wrap a chosen
charting library behind a thin presentational component; pages own data + state.

### Services / state (`features/analytic/services/`)

- `analytic.service` wrapping the core `api.service`; typed request params
  (range, granularity, dimension, filters) and typed metric/series/report models in
  `features/analytic/models/`.
- View state via signals: selected date range, active filters, granularity; async
  data via RxJS streams bridged with `toSignal`. Immutable updates only.

### Guards

- `auth.guard` + `role.guard` (`admin`) — the backend gates the analytics
  dashboard/metrics/reports routes as `admin`-only, so these screens are admin-scoped.

## Consumed API

All under `/api/v1`, envelope `{ success, data, error, meta }` (paginated lists carry
`meta`). Bind to the backend `analytic` domain:

- `GET /analytics/dashboard` — dashboard aggregate / headline KPI metrics (`admin`).
- `GET /analytics/metrics` — rolled-up metrics (typed metric set).
- `GET /analytics/events` — paginated raw events (`?page&limit` + filters).
- `GET /analytics/reports/dau` · `GET /analytics/reports/mau` — DAU / MAU reports.

Register every consumed endpoint in `context/api-context.md`.

## UI / components used

- Shared primitives (card, button, input) + the new analytic components.
- Token-only styling for charts (series colors from tokens / accent ramp); every
  async surface implements loading / empty / error / success; OnPush throughout.
- Numbers/dates localized via the i18n service.

## Acceptance criteria

- [ ] `/analytics` renders KPI metric cards from `GET /analytics/dashboard` (and/or
      `GET /analytics/metrics`), with loading / empty / error / success states.
- [ ] Time-series and breakdown charts render from `GET /analytics/metrics` /
      `GET /analytics/reports/{dau,mau}` and react to the date-range + filter controls.
- [ ] `/analytics/reports` lists paginated rows from `GET /analytics/events` driven by
      `meta`; sorting + filters update the query and refetch.
- [ ] Date-range / granularity / filter state held in signals; updates are immutable;
      data fetched via RxJS bridged to signals.
- [ ] All requests/responses typed against `features/analytic/models`; components never
      call `HttpClient`; consumed endpoints registered in `api-context.md`.
- [ ] Routes guarded by `auth.guard` + `role.guard` (admin only; `403` handled).
- [ ] Chart series colors and all visuals come from design tokens (no hardcoded hex);
      numbers/dates localized.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- left for /parse-features -->
