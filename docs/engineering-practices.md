# Engineering Practices

> Practices applied throughout the delivery of a commercial SaaS web
> application. Every item below is grounded in observable evidence from the
> delivered codebase. No proprietary code is reproduced.

These practices reflect the standards expected in the German software
engineering market: correctness, clarity, maintainability, and accountability.

---

## 1. Clean Architecture & Separation of Concerns

- A strict, downward-only dependency direction:
  **presentation → state → service → transport**.
- UI components never issue network requests directly; all access flows through
  a feature service and a shared transport layer.
- Business-agnostic utilities (formatting, parsing, request helpers) are isolated
  from feature code under `utils/`.

**Why it matters:** changes to a cross-cutting concern (e.g. how auth headers are
attached) happen in exactly one place.

---

## 2. SOLID-Aligned Design

| Principle | How it shows up |
|-----------|-----------------|
| **Single Responsibility** | One feature folder owns one slice + one API module. |
| **Open/Closed** | New features are added as new service folders without modifying existing ones. |
| **Liskov / Interface segregation** | Small, focused request helpers (`get`, `post`, multipart, streaming) rather than one mega-client. |
| **Dependency Inversion** | Features depend on the abstract request helpers, not on Axios directly. |

---

## 3. Type Safety

- **TypeScript `strict` mode** is enabled project-wide.
- **Schema-derived types**: Zod schemas are the single source of truth, with
  TypeScript types *inferred* from them (`z.infer<...>`).
- **Typed Redux access**: thin typed wrappers (`useAppSelector`,
  `useAppDispatch`) ensure store access is fully typed at every call site.
- **Centralized enums** for endpoints, HTTP status, auth status, and roles avoid
  magic strings and typos.

---

## 4. Reusable Components

- A dedicated **UI primitive library** (`components/ui/`) provides accessible,
  composable building blocks (buttons, dialogs, tables, forms, tabs, etc.) built
  on Radix primitives.
- **Variant-driven styling** via `class-variance-authority` plus `clsx` and
  `tailwind-merge` keeps styling declarative and conflict-free.
- Feature components compose these primitives rather than re-implementing UI.

See [`../examples/reusable-component-patterns.md`](../examples/reusable-component-patterns.md).

---

## 5. Code Review Culture & Pull-Request Workflow

- Work is integrated through **pull requests into a protected `main` branch**.
- CI runs on push to `main`, making the pipeline a gate on what reaches
  production.
- Module-level comment banners and JSDoc on shared utilities make diffs readable
  and reviewable.

---

## 6. Git & Branching Strategy

- **Trunk-style flow** with `main` as the production branch.
- Deployment is **triggered by merges to `main`** (and on-demand via manual
  workflow dispatch), keeping the deployed state traceable to Git history.

---

## 7. Documentation Standards

- Consistent **module banner comments** delineate sections within larger files.
- **JSDoc** documents shared hooks and helpers.
- Configuration (environments, build scripts) is self-documenting through clearly
  named npm scripts (`dev:staging`, `build:staging`, `build:production`,
  `start:production`).

---

## 8. Dependency Management

- Dependencies are **explicitly versioned** in `package.json` with a committed
  lockfile for reproducible installs.
- **`engines` constraints** pin supported Node and npm versions, preventing
  "works on my machine" drift.
- Tooling dependencies (ESLint, Prettier, TypeScript) are kept current and
  aligned with the framework's recommended configurations.

---

## 9. Environment Segregation

- Separate environment files for **staging** and **production**.
- `dotenv-cli` injects the correct environment per npm script — there is no
  ambiguity about which configuration a build uses.
- Public configuration (API base URL, payment publishable key, socket URL) is
  provided via environment variables; **secrets are never hard-coded**.

---

## 10. Defensive Programming

Concrete examples observed in the codebase:

- **SSR-safe persistence**: a noop storage implementation prevents persistence
  from crashing during server rendering.
- **Resilient authentication**: single-flight token refresh with automatic retry,
  and a clean session teardown + redirect when refresh is impossible.
- **Dual auth-failure detection**: handles both HTTP `401` and APIs that embed
  auth failures inside a `200` payload.
- **Network-error guards**: requests distinguish connectivity failures from
  application errors.
- **Serialization safety**: non-serializable values are explicitly excluded from
  Redux's serializability checks rather than disabling the safeguard.

---

## 11. Developer Experience

- **Turbopack** for fast local development.
- **Path aliasing** (`@/*`) for clean, refactor-friendly imports.
- **Prettier-as-lint-error** guarantees a single, non-negotiable code style.
- **React Query Devtools** wired in for inspecting server-state during
  development.

---

## Summary

The codebase optimizes for the long game: predictable structure, strong typing,
centralized cross-cutting logic, and a delivery pipeline that ties production
state directly to reviewed Git history.
