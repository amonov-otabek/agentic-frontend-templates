# Feature 03 — Analytics Dashboard (Frontend)

- Feature ID: FEAT-03
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — this is a guarded dashboard surface)

> Source of truth for the authenticated **analytics** surface: a summary dashboard plus a
> paginated event explorer with charts. Consumes the backend `analytic` domain.

## Goal

Give authenticated users an analytics overview on `/analytics`: headline stat cards, a
time-series chart of events, and a paginated event list — all read through TanStack Query
against the backend analytic API.

## Scope

- **Analytics page** (`features/analytic/pages/analytics-page`, route `/analytics` under
  `DashboardLayout`) — orchestrates the summary + chart + event list.
- **Summary stat cards** (`components/stat-cards`) — headline counts from the dashboard summary.
- **Events chart** (`components/events-chart`) — time-series visualization built with
  `recharts` (3.x) using the chart token ramp (`--chart-1…5`); wraps `shared/components/ui/chart`.
- **Event list** (`components/event-list`) — paginated table of analytics events
  (`?page&limit`), using `shared/hooks/use-pagination` + the `pager` widget.
- **Queries** (`queries/analytics.queries`) — `useDashboard` (summary) and `useEvents`
  (paginated); typed via `types/analytics.types` (`DashboardSummary`, `AnalyticsEvent`,
  `EventListQuery`).
- Every panel handles loading (skeleton) / error / empty / success states.
- Out of scope: writing/emitting analytics events (server-side), CSV export, custom date-range pickers.

## Consumed API

Base `/api/v1/analytics`, envelope `{ success, data, meta? }`.

- `GET /analytics/dashboard` → `DashboardSummary` (headline metrics).
- `GET /analytics/events?page&limit` → paginated `AnalyticsEvent[]` with `meta`.

## UI / components used

- `features/analytic/components/{stat-cards,events-chart,event-list}`.
- shadcn `card`, `chart`, `table`, `skeleton`, `empty`, `badge`; widgets `pager`;
  `recharts`; `shared/hooks/use-pagination`; `DashboardLayout`.

## Acceptance criteria

- [ ] `/analytics` renders behind the auth guard (unauthenticated → `/sign-in`).
- [ ] Stat cards render headline metrics from `GET /analytics/dashboard`.
- [ ] The events chart renders a time series with `recharts` using only chart tokens
      (`--chart-1…5`) — no hardcoded colors.
- [ ] The event list paginates via `?page&limit` and reads counts from `meta`
      (`total/page/limit`); the pager reflects `totalPages`.
- [ ] Loading (skeleton), error, and empty states render on every panel.
- [ ] Data flows through `queries/analytics.queries` + `api/analytics.api` only — no
      `fetch` in components; responses typed via `types/analytics.types`.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
