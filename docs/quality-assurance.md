# Quality Assurance

> The quality practices applied during delivery of a commercial SaaS web
> application. Described at the level of *approach and tooling*, inferred from the
> delivered codebase.

---

## 1. Static Quality Gates

The first line of defense runs before any code executes:

- **TypeScript `strict` mode** — eliminates entire classes of runtime errors
  (null/undefined access, implicit `any`, unsafe narrowing) at compile time.
- **ESLint** — configured with Next.js `core-web-vitals` and TypeScript rule sets,
  plus the TanStack Query plugin to catch server-state misuse.
- **Prettier enforced as an error** — formatting is not a matter of opinion;
  non-conforming code fails linting, keeping diffs noise-free.

---

## 2. Type-Driven Correctness

- **Schema-first validation**: Zod schemas define and validate all form/input
  contracts, and TypeScript types are inferred from them — the validator and the
  type can never drift apart.
- **Typed store access** via `useAppSelector` / `useAppDispatch` ensures state
  reads and dispatches are checked at compile time.
- **Centralized enums** for endpoints, statuses, and roles remove stringly-typed
  errors.

---

## 3. Manual Validation & Regression

- Features were validated across the distinct **role surfaces** (user, admin,
  operator) before merge, since access and capability differ per role.
- **Staging-first promotion** provides a production-like environment for
  regression checks before customer-facing release.
- The pull-request workflow ensures a **second pair of eyes** on every change
  prior to integration.

---

## 4. Release Verification

| Stage | Check |
|-------|-------|
| Pre-merge | Lint + type check, code review |
| Staging | Functional/regression validation in a production-like environment |
| CI build | Production build must succeed to deploy |
| Deploy | Only reviewed `main` commits reach production |

A failing build is a **hard gate**: it blocks deployment rather than producing a
partial release.

---

## 5. Resilience as a Quality Attribute

Quality isn't only "does it work in the happy path" — the codebase encodes
behavior for the unhappy paths:

- Authentication expiry → transparent refresh and retry, or clean logout.
- Network failure → distinguished from application errors, surfaced to the user.
- SSR constraints → storage fallbacks so rendering never crashes.
- Non-serializable state → explicitly handled rather than ignored.

---

## 6. Monitoring & Crash Reporting (Posture)

> Stated honestly to the level supported by evidence.

- User-facing failures are surfaced through a **centralized notification layer**,
  giving consistent, observable error feedback in the UI.
- The hosting platform (Azure Static Web Apps) provides **deployment-level
  observability** and the ability to roll back to a prior known-good deploy.
- Dedicated third-party crash/error-reporting and analytics integrations are an
  **organizational/ops concern**; this document does not claim a specific tool
  beyond what the front-end codebase demonstrates.

---

## 7. Continuous Improvement

- **Devtools integration** (React Query Devtools) supports diagnosing
  server-state behavior during development.
- **Fast feedback loops** via Turbopack keep iteration cheap, encouraging small,
  verifiable changes over large risky ones.

---

## Summary

Quality on this engagement was enforced *structurally* — through strict typing,
schema-derived contracts, error-level formatting, a review-gated pipeline, and
staging-first promotion — rather than relying on manual diligence alone.
