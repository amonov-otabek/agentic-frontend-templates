# Feature 04 — Audit Log Viewer (Frontend)

- Feature ID: FEAT-04
- Status: draft
- Depends on: FEAT-01 (scaffold), FEAT-02 (auth + guarded dashboard)

> Source of truth for the authenticated **audit log + security alerts** viewer.
> `/parse-features 04` turns the `## Tasks` section into
> `context/tasks-list/04-audit-frontend-tasks.md`.

## Goal

Let authorized users browse the backend audit trail and triage security alerts:
a filterable, paginated audit-log table plus an alerts table with a resolve
action, themed and localized.

## Scope (real pages, routes, components, composables)

- **Page** (`src/features/audit/pages/audit-page.vue`): mounted at `/audit` under
  `DashboardLayout` (`requiresAuth`, `meta.titleKey:'nav.audit'`).
- **Components** (`src/features/audit/components/`): `audit-log-table.vue`
  (filterable by user/action/entity, paginated), `alert-table.vue` (severity +
  resolved filters, resolve action).
- **Queries** (`queries/audit.queries.ts`): `useAuditLogs(query)`,
  `useAlerts(query)`, `useResolveAlert()`.
- **Types** (`types/audit.types.ts`): `AuditLog`, `SecurityAlert`,
  `AuditListQuery`, `AlertListQuery`.
- **RBAC**: gate audit access to elevated roles via `use-permission` (backend
  authoritative).
- Out of scope: log export and per-entry detail drawer beyond row expansion (flag
  if scoped).

## Consumed API (`/api/v1/audit`, shared envelope)

- `GET /audit?page&limit&userId&action&entity` → `Paginated<AuditLog>`.
- `GET /audit/alerts?page&limit&severity&resolved` → `Paginated<SecurityAlert>`.
- `PATCH /audit/alerts/:id/resolve` → `SecurityAlert`.

## UI / components used

- shadcn-vue `table`, `select`/`combobox` (filters), `badge` (severity),
  `button`, `skeleton`, `empty`, `pager`/`use-pagination`, `sonner`.

## Acceptance criteria

- [ ] `/audit` is guarded and reachable from the dashboard; restricted to elevated roles.
- [ ] Audit log table paginates and filters by user/action/entity against the API.
- [ ] Alerts table filters by severity/resolved and renders severity badges.
- [ ] Resolve action calls `PATCH /audit/alerts/:id/resolve`, updates the row, and surfaces success/error.
- [ ] Loading/error/empty states on both tables; labels localized in en/ru/uz; errors keyed on `error.code`.
- [ ] Tokens-only styling, a11y, lint, typecheck, build, and tests pass.

## Tasks
