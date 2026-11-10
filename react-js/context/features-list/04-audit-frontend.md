# Feature 04 — Audit Log & Alerts (Frontend)

- Feature ID: FEAT-04
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth — guarded dashboard surface)

> Source of truth for the authenticated **audit** surface: a filterable audit-log viewer
> and a security-alerts panel with resolve actions. Consumes the backend `audit` domain.

## Goal

Give authenticated (admin) users an audit surface on `/audit`: browse the paginated audit
log with filters, review security alerts by severity/resolution, and resolve open alerts —
all through TanStack Query against the backend audit API.

## Scope

- **Audit page** (`features/audit/pages/audit-page`, route `/audit` under `DashboardLayout`)
  — orchestrates the log table and the alerts panel (e.g. via tabs).
- **Audit-log table** (`components/audit-log-table`) — paginated log with filters
  (`userId`, `action`, `entity`); columns for actor, action, entity, timestamp.
- **Alert table** (`components/alert-table`) — paginated security alerts with `severity`
  + `resolved` filters and a per-row Resolve action.
- **Queries** (`queries/audit.queries`) — `useAuditLogs`, `useAlerts`, `useResolveAlert`
  (mutation invalidates the alerts list); typed via `types/audit.types` (`AuditLog`,
  `SecurityAlert`, `AuditListQuery`, `AlertListQuery`).
- Loading / error / empty states on both tables; resolve gives optimistic/toast feedback.
- Out of scope: writing audit entries (server-side), CSV export, alert-rule configuration.

## Consumed API

Base `/api/v1/audit`, envelope `{ success, data, meta? }`.

- `GET /audit?page&limit&userId&action&entity` → paginated `AuditLog[]` with `meta`.
- `GET /audit/alerts?page&limit&severity&resolved` → paginated `SecurityAlert[]`.
- `PATCH /audit/alerts/:id/resolve` → updated `SecurityAlert`.

## UI / components used

- `features/audit/components/{audit-log-table,alert-table}`.
- shadcn `table`, `card`, `tabs`, `badge`, `select`/`native-select`, `button`,
  `skeleton`, `empty`, `sonner`; widgets `pager`; `shared/hooks/usePagination`;
  `shared/hooks/usePermission` (gating); `DashboardLayout`.

## Acceptance criteria

- [ ] `/audit` renders behind the auth guard (unauthenticated → `/sign-in`) and is
      role-gated via `usePermission`.
- [ ] The audit-log table paginates and filters by `userId` / `action` / `entity`.
- [ ] The alert table paginates and filters by `severity` / `resolved`.
- [ ] Resolving an alert calls `PATCH /audit/alerts/:id/resolve`, invalidates the alerts
      query, and surfaces success/error via toast.
- [ ] Loading (skeleton), error, and empty states render on both tables.
- [ ] All HTTP flows through `api/audit.api` + `queries/audit.queries`; responses typed
      via `types/audit.types`; no `fetch` in components.
- [ ] All labels localized (en/ru/uz).
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
