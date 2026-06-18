# Feature 03 — Analytics Dashboard (Frontend)

- Feature ID: FEAT-03
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth + guarded dashboard)

> Source of truth for the authenticated **analytics dashboard**. `/parse-features 03`
> turns the `## Tasks` section into `context/tasks-list/03-analytic-frontend-tasks.md`.
> Builds on the dashboard shell and API client.

## Goal

Give authenticated users an analytics overview: summary stat cards plus an events
chart and a paginated event list, served by the backend analytics domain and
rendered with `@unovis/vue`, themed and localized.

## Scope (real pages, routes, components, composables)

- **Page** (`src/features/analytic/pages/analytics-page.vue`): mounted at
  `/analytics` under `DashboardLayout` (`requiresAuth`, `meta.titleKey:'nav.analytics'`).
- **Components** (`src/features/analytic/components/`): `stat-cards.vue`
  (summary KPIs), `events-chart.vue` (`@unovis/vue` time-series), `event-list.vue`
  (paginated table).
- **Queries** (`queries/analytics.queries.ts`): `useDashboard()`,
  `useEvents(query)` (page/limit reactive via `MaybeRefOrGetter`).
- **Types** (`types/analytics.types.ts`): `DashboardSummary`, `AnalyticsEvent`,
  `EventListQuery`.
- Out of scope: report export / custom date-range builders (flag if scoped).

## Consumed API (`/api/v1/analytics`, shared envelope)

- `GET /analytics/dashboard` → `DashboardSummary` (stat cards).
- `GET /analytics/events?page&limit` → `Paginated<AnalyticsEvent>` (chart + list).

## UI / components used

- shadcn-vue `card`, `table`, `skeleton`, `badge`; `@unovis/vue` chart;
  `pager`/`use-pagination` for the event list; `sonner` for errors.

## Acceptance criteria

- [ ] `/analytics` is guarded and reachable from the dashboard sidebar.
- [ ] Stat cards render `GET /analytics/dashboard` with loading/error/empty states.
- [ ] Events chart renders the events series via `@unovis/vue` using theme tokens (no hardcoded colors).
- [ ] Event list paginates `GET /analytics/events` with `meta`-driven pager and loading/empty states.
- [ ] All labels localized in en/ru/uz; errors surfaced via `ApiError`/toast and keyed on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
