# Project Standards

Implementation rules for all code. This is the **Next.js web client**. Treat as locked.
UI specifics live in `context/ui-rules.md` + `context/ui-context.md`; this file governs all code.

## Engineering mindset

1. **Think first** — read the task block and relevant context before writing code.
2. **Scope is sacred** — build what the task says; nothing more, nothing less.
3. **Every change is verifiable** — if it can't be seen working or typed/built, it isn't done.
4. **Clean over clever** — boring, readable code wins.
5. **One thing at a time** — never mix refactoring with feature work.
6. **Failures are expected** — handle loading/empty/error states as first-class flow.
7. **Fix root causes** — never patch symptoms or wrap broken code.

## TypeScript

- Strict mode (`strict: true`). No `any` — use precise types or `unknown` + narrowing.
  No `as` to silence type errors.
- ES2017 target, `moduleResolution: bundler`, `jsx: react-jsx`. Use the `@/*` path alias
  for cross-area imports; keep feature-internal imports relative within the slice.
- Explicit return types on exported functions and hooks.
- DTOs/response types live in the feature's `types/` and mirror the backend shapes — define
  once, import everywhere; never restate a shape by hand.

## React / Next idioms

- **Server Components by default.** Add `"use client"` only where you need hooks, event
  handlers, TanStack Query, Zustand, or browser APIs.
- **Pages orchestrate, components render.** `app/**/page.tsx` is a thin entry that renders a
  feature `pages/` component. A `pages/` component composes `components/`; it never inlines
  section markup.
- **No business logic in components.** Components take props and render; logic lives in
  `services/`, data in `queries/`, state in `store/`.
- Static lists (nav, footer) live inside the component as explicit JSX — pages pass only
  dynamic data (no `.map()` over static literal arrays).
- Hooks follow rules-of-hooks; custom hooks live in `shared/hooks/` or a feature when slice-local.

## Data fetching

- All HTTP goes through the typed client (`shared/libs/api.ts`) from a feature's `api/` folder
  (`api.get/getPaginated/post/put/patch/delete`). No bare `fetch` in components/pages.
- Server state is TanStack Query only. Each feature defines a **query-key factory** and uses it
  for `useQuery`/`useMutation`; mutations invalidate the affected keys on success.
- Paginated lists use `api.getPaginated` and the `meta { total, page, limit }` it returns.
- Never store server data in Zustand; Zustand is for client/UI/session state.

## Forms, responses & errors

- Validate input with Zod at the form boundary using `shared/libs/validators.ts`
  (`validate`, `formatZodErrors`) plus the feature's `validators/` schemas.
- The client unwraps the envelope and throws a typed `ApiError(status, code, message, fields)`.
  Handle failures explicitly; map `error.fields` back to form fields and assert on
  `error.code` (errors are i18n-localized — never branch on message text).
- Every async surface renders all states: loading, empty, error, success. No silent failures,
  no empty catch blocks.
- Surface user-facing failures via `sonner` toasts or inline field errors — never leak raw errors.

## State & immutability

- Zustand stores via `shared/libs/store.ts` (`create` + `createSelectors`); read fields through
  the generated `.use.field()` selectors.
- Never mutate state, props, or query data — return new objects (spread). Immutability everywhere.

## Styling

- Tokens only: every color/size/radius/spacing comes from a token in `context/ui-context.md`
  (CSS vars in `styles/globals.css`). Zero hardcoded hex/px/magic numbers — a QA rejection.
- Compose classes with `cn` (`shared/libs/utils.ts`); reuse `shared/components/ui` primitives.
  Do not add a second component or icon library.

## Accessibility

- Semantic HTML first; labels on inputs; visible focus styles; full keyboard reachability;
  text contrast ≥ 4.5:1; touch targets ≥ 44×44px. `aria` only where semantics can't express it.

## i18n

- All user-facing strings resolve through `next-intl` (`getTranslations` / `useTranslations`);
  no hardcoded copy. Keep `messages/{en,ru,uz}.json` in sync; URLs stay locale-prefixed.

## Security

- Config via `NEXT_PUBLIC_*` env only; never hardcode URLs or secrets. Note that anything
  `NEXT_PUBLIC_*` ships to the browser — keep real secrets server-side.
- JWT bearer is attached by the client; never log tokens, passwords, or PII.

## Testing

- **Where tests live (HARD):** every test file goes in `test/` (`main`, `app`, `setup`, `helpers`, `e2e/`), at the path that
  mirrors its source file — see `context/architecture.md` → **Test structure**. Never beside a
  source file, never inside a module/feature folder. `setup` keeps the suite
  self-contained: tests run on a clean checkout with no developer `.env`.
- New behavior ships with at least a happy-path check plus one edge/error case where a test
  harness exists; pre-existing checks stay green. The `next build` compile gate must pass.

## Comments, TODOs, dependencies

- Comments explain *why*, not *what*. No TODO/FIXME/HACK without a linked task.
- Prefer existing dependencies; a new one requires developer approval (note it in QA notes).
- Before using any third-party library, verify against the installed version's docs — APIs drift.
- pnpm is the package manager (see `context/architecture.md`); do not introduce a second lockfile.
