# Project Standards

Implementation rules for all code. Derived from the `vue-js` reference. Frontend SPA
project — read alongside `context/ui-rules.md` for visual rules. Treat as locked.

## Engineering mindset

1. **Think first** — read the task block and relevant context before writing code.
2. **Scope is sacred** — build what the task says; nothing more, nothing less.
3. **Every change is verifiable** — if it can't be tested or observed, it isn't done.
4. **Clean over clever** — boring, readable code wins.
5. **One thing at a time** — never mix refactoring with feature work.
6. **Failures are expected** — handle loading/empty/error states as first-class flow.
7. **Fix root causes** — never patch symptoms or wrap broken code.

## TypeScript

- Strict mode (via `@vue/tsconfig`); `noFallthroughCasesInSwitch` on. Unused-locals are
  enforced by ESLint for app code (tsconfig relaxes them for vendored shadcn-vue).
- No `any` — use precise types or `unknown` + narrowing. No `as` to silence errors.
- Use the `@/*` path alias — not deep relative `../../..` chains.
- DTOs and view models live in `features/<name>/types`; shared types in `shared/libs`.
- Validate all external/form input with Zod at the boundary (`shared/libs/validators`
  + feature `validators/`).

## Vue component rules

- **Composition API + `<script setup lang="ts">`** for every component. No Options API.
- **Pages orchestrate, components render.** A page (`pages/*.vue`) composes feature
  components and wires queries; components (`components/*.vue`) take typed props,
  `emit` typed events, and render. No business/data logic in templates.
- `defineProps`/`defineEmits` are typed with interfaces — no runtime prop objects when
  a type works. Provide sensible `withDefaults`.
- Reusable logic goes in composables (`shared/hooks/use-*.ts` or feature-local) — not
  copy-pasted across components.
- Required interaction states for interactive elements (default, hover, focus-visible,
  disabled, loading) and data states for async (loading, empty, error, success).

## Data fetching & state

- **Server state via TanStack Query only.** Read with `useQuery`, write with
  `useMutation`, declared in `queries/*.queries.ts` with stable, namespaced query keys
  (e.g. `authKeys`). Invalidate/`setQueryData` on mutation success — never refetch by hand.
- **HTTP through the api client only** (`shared/libs/api.ts`). Feature `api/*.ts` are
  the only files that name backend paths. Components never call `fetch` or `api` directly.
- **Client state via Pinia setup stores** (`features/<name>/store`), defined with
  `defineStore` from `shared/libs/store`. Mutate through actions; never assign store
  refs from outside.
- Treat all data as immutable — return new objects/arrays; never mutate props or
  cached query data in place.

## Responses & errors

- The api client unwraps the envelope and throws a typed `ApiError`
  (`status`, `code`, `fields?`). Handle failures explicitly: map `error.fields` to form
  field errors and surface a user-friendly message via `useToast`.
- Never swallow errors silently; never render a raw error object/stack to the user.
- Assert on `error.code` (stable) rather than message text (i18n-localized).

## Forms & validation

- Forms use vee-validate + `@vee-validate/zod`; the Zod schema is the single source of
  truth for shape and messages. Reuse shared schemas (`emailSchema`, `passwordSchema`,
  `paginationQuerySchema`) from `shared/libs/validators`.
- Disable submit controls while the mutation is pending.

## i18n

- All user-facing strings resolve through `vue-i18n` (`shared/i18n`, locales en/ru/uz).
  No hardcoded copy in templates; add keys to every locale file.

## Styling

- Tokens only (see `context/ui-context.md`); compose classes with `cn()`
  (`shared/libs/utils`). No inline hex/px, no second UI/icon library.

## Testing

- **Where tests live (HARD):** every test file goes in `test/` (`main`, `app`, `setup`, `helpers`, `e2e/`), at the path that
  mirrors its source file — see `context/architecture.md` → **Test structure**. Never beside a
  source file, never inside a module/feature folder. `setup` keeps the suite
  self-contained: tests run on a clean checkout with no developer `.env`.
- New behavior ships with at least a happy-path check plus one edge/error case
  (component or composable level). Pre-existing checks stay green; the `pnpm build`
  (`vue-tsc`) gate must pass — a task that breaks it is not done.

## Comments, TODOs, dependencies

- Single-line comments only; explain *why*, not *what*. No TODO/FIXME/HACK without a
  linked task.
- Prefer existing dependencies; a new one requires developer approval (note it in QA
  notes). Before using a library, verify against its installed version's docs.
- pnpm is the package manager (see `context/architecture.md`); do not introduce a
  second lockfile.
