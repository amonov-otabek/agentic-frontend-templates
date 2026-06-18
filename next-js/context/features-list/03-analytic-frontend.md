# Feature 03 — Analytics Dashboard (Frontend)

- Feature ID: FEAT-03
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01, 02

> Source of truth for the **analytics** feature slice — a guarded dashboard that
> summarizes activity (stat cards + chart) and lists the raw event stream from the
> backend analytic domain. `## Tasks` below is left for `/parse-features`.

## Goal

An authenticated analytics overview: at-a-glance summary metrics, a trend chart,
and a paginated event list — fetched through TanStack Query and rendered with
`recharts`, in the dashboard shell.

## Scope

- **Route:** `src/app/[locale]/(protected)/analytics/page.tsx` (thin) → renders
  `features/analytic/pages/analytics-page.tsx`.
- **Summary:** `stat-cards.tsx` renders the dashboard summary metrics
  (`DashboardSummary`).
- **Chart:** `events-chart.tsx` renders an events trend via `recharts`
  (`shared/components/ui/chart`).
- **Event list:** `event-list.tsx` paginated table of `AnalyticsEvent` rows with
  `use-pagination` + `pager`.
- **Queries:** `queries/analytics.queries.ts` — `useDashboard`, `useEvents(query)`
  with a key factory; list invalidation on relevant mutations (read-mostly slice).
- **Types:** `types/analytics.types.ts` — `DashboardSummary`, `AnalyticsEvent`,
  `EventListQuery`.
- **Out of scope:** event ingestion/writing (backend-owned); custom report builder.

## Consumed API

Base `${NEXT_PUBLIC_API_URL}/api/v1`, envelope; called only from
`features/analytic/api/analytics.api.ts`.

- `GET /analytics/dashboard` → `DashboardSummary`
- `GET /analytics/events?page&limit` → paginated `AnalyticsEvent[]` (`meta`)

## UI / components used

- `features/analytic/components/{stat-cards,events-chart,event-list}.tsx`.
- Shared UI: `card`, `chart` (recharts wrapper), `table`, `skeleton`, `empty`,
  `pagination`/`pager`; `use-pagination` hook; dashboard shell from `(protected)`.

## Acceptance criteria

- [ ] Analytics route is guarded (unauthenticated → sign-in) and renders inside the
      dashboard shell.
- [ ] Stat cards render `DashboardSummary` from `GET /analytics/dashboard`.
- [ ] Events chart renders the trend via `recharts` from the events data.
- [ ] Event list paginates against `GET /analytics/events` using `meta`
      (total/page/limit) + `use-pagination`.
- [ ] Loading / empty / error states handled on each panel; errors branch on
      `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
