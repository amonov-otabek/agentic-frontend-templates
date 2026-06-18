# Project Standards

Implementation rules for all code. This is the **Vite + React SPA** client for
`express-monolith-api`. Treat as locked.

## Engineering mindset

1. **Think first** — read the task block and relevant context before writing code.
2. **Scope is sacred** — build what the task says; nothing more, nothing less.
3. **Every change is verifiable** — if it can't be typechecked, built, or seen in the
   UI, it isn't done.
4. **Clean over clever** — boring, readable components win.
5. **One thing at a time** — never mix refactoring with feature work.
6. **Failures are expected** — handle loading, empty, and error states as first-class.
7. **Fix root causes** — never patch symptoms or wrap broken code.

## TypeScript

- Strict mode on (`noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch`,
  `verbatimModuleSyntax`, `erasableSyntaxOnly`).
- No `any` — use precise types or `unknown` + narrowing. No `as` to silence errors.
- Use the `@/*` alias — not deep relative `../../..` chains.
- Use `import type { … }` for type-only imports (required by `verbatimModuleSyntax`).
- Explicit return types on exported functions and API/service modules.
- Validate all external/user input (forms, params, query) with Zod at the boundary.

## React idioms

- Function components only; hooks at the top level (Rules of Hooks). No class
  components.
- **Pages orchestrate, components render.** Pages call query/mutation hooks and compose
  feature components; components receive typed props and render — no fetching inside.
- No business logic in components — derive/compute in hooks, services, or the page.
- Keep effects minimal; prefer derived state and Query over `useEffect` data loading.
- Lists need stable `key`s; never index unless the list is static and never reordered.
- Static lists (nav, footer) live inside the component as explicit JSX, not `.map()` of
  literals passed as props.

## Layered feature rules

- **api/** — thin functions over `shared/libs/api` (`api.get/post/...`). One file per
  backend resource; no React, no JSX. All HTTP lives here.
- **queries/** — TanStack Query hooks + a query-key factory. `useQuery` for reads,
  `useMutation` (with `invalidateQueries`) for writes. No `fetch` here.
- **types/** — TS interfaces mirroring backend models + DTOs. No behavior.
- **components/** — presentational, prop-driven, render-only.
- **pages/** — route entries: call hooks, handle data states, compose components.
- **validators/** — Zod schemas, one per form/input surface.

## Data fetching

- Server state goes through TanStack Query — never store fetched data in Zustand or
  component state long-term.
- Use the shared query-key factory per feature; mutations invalidate the right keys.
- Lists use `getPaginated` and read `data` + `meta` ({ total, page, limit }).
- Surface every async state: loading (skeleton/spinner), empty, error, success.
- Errors are `ApiError` instances — branch on `error.code`/`error.fields`, not message
  text (messages are i18n-localized server-side).

## Responses & errors

- All API access returns unwrapped `data` (or `{ data, meta }`) from `shared/libs/api`;
  never re-implement envelope parsing in a component.
- Show user-facing errors via `sonner` toasts or inline field errors from
  `error.fields`; never render a raw exception or stack.
- No empty catch blocks; expected failures (validation, 401, not-found) are flow.

## Client state

- Use Zustand (`shared/libs/store.ts`) only for genuinely client-side state (auth user,
  UI toggles). Access via the generated `.use.<field>()` selectors to avoid re-renders.
- Never mutate state — return new objects/arrays (immutable updates everywhere).

## Styling & UI

- Tokens only — every visual value from `styles/globals.css` / `context/ui-context.md`.
  No hardcoded hex, px font sizes, or magic spacing. See `context/ui-rules.md`.
- Reuse `shared/components/ui` (shadcn) primitives; do not hand-fork them. Compose with
  `cn()` (`shared/libs/utils`).
- Check the component registry in `context/ui-context.md` before building a component;
  register any new one.

## Accessibility

- Semantic HTML first; labels on inputs; alt text on images; aria only where semantics
  can't express it. Visible focus styles; full keyboard reachability.
- Text contrast ≥ 4.5:1; touch targets ≥ 44×44px; controls disabled while their async
  action runs.
- Respect `prefers-reduced-motion`.

## i18n

- All user-facing strings resolve through `react-i18next` (`t("key")`) with keys in
  `shared/i18n/messages/{en,ru,uz}.json` — no hardcoded display text.

## Testing

- New behavior ships with a happy-path test plus at least one edge/error case
  (component/hook tests; Playwright for critical e2e flows).
- Assert on `error.code`, not localized message text.
- Pre-existing tests stay green; a task that breaks them is not done.

## Comments, TODOs, dependencies

- Comments explain *why*, where the code can't. No TODO/FIXME/HACK without a linked
  task. Preserve `eslint-disable` / `@ts-*` only where load-bearing.
- Prefer existing dependencies; a new one requires developer approval (note it in QA
  notes). Before using any library, verify against the installed version's docs.
- pnpm is the package manager (see `context/architecture.md`); do not add a second
  lockfile.
