# Feature 04 — Audit Log & Alerts (Frontend)

- Feature ID: FEAT-04
- Status: draft   <!-- /parse-features REFUSES to run until you change this to: ready -->
- Depends on: 01, 02

> Source of truth for the **audit** feature slice — an admin-gated viewer over the
> backend audit domain: a filterable audit-log table and a security-alerts table
> with resolve. `## Tasks` below is left for `/parse-features`.

## Goal

An authenticated, RBAC-gated audit surface: browse the audit log (filter by user /
action / entity) and triage security alerts (filter by severity / resolved, mark
resolved) — paginated and rendered in the dashboard shell.

## Scope

- **Route:** `src/app/[locale]/(protected)/audit/page.tsx` (thin) → renders
  `features/audit/pages/AuditPage.tsx`.
- **Audit log table** (`AuditLogTable.tsx`): paginated `AuditLog` rows with
  filters (`userId`, `action`, `entity`).
- **Alerts table** (`AlertTable.tsx`): paginated `SecurityAlert` rows with filters
  (`severity`, `resolved`) and a per-row resolve action.
- **Queries:** `queries/audit.queries.ts` — `useAuditLogs(query)`,
  `useAlerts(query)`, `useResolveAlert` (invalidates the alerts key on success).
- **Types:** `types/audit.types.ts` — `AuditLog`, `SecurityAlert`,
  `AuditListQuery`, `AlertListQuery`.
- **RBAC:** gated by `usePermission` (admin) + the `(protected)` guard.
- **Out of scope:** audit-event writing (backend-owned); alert configuration.

## Consumed API

Base `${NEXT_PUBLIC_API_URL}/api/v1`, envelope; called only from
`features/audit/api/audit.api.ts`.

- `GET /audit?page&limit&userId&action&entity` → paginated `AuditLog[]`
- `GET /audit/alerts?page&limit&severity&resolved` → paginated `SecurityAlert[]`
- `PATCH /audit/alerts/:id/resolve` → `SecurityAlert`

## UI / components used

- `features/audit/components/{audit-log-table,alert-table}.tsx`.
- Shared UI: `table`, `badge`, `select`/`native-select` (filters), `button`,
  `dialog`/`alert-dialog` (resolve confirm), `pagination`/`pager`, `empty`,
  `skeleton`; `usePagination`, `usePermission`, `use-toast` hooks.

## Acceptance criteria

- [ ] Audit route is guarded and admin-gated (`usePermission`); renders in the
      dashboard shell.
- [ ] Audit-log table paginates against `GET /audit` and filters by user / action /
      entity.
- [ ] Alerts table paginates against `GET /audit/alerts` and filters by severity /
      resolved.
- [ ] Resolving an alert calls `PATCH /audit/alerts/:id/resolve` and invalidates the
      alerts query (row reflects resolved state).
- [ ] Loading / empty / error states handled; errors branch on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks

<!-- Left for /parse-features to decompose. -->
