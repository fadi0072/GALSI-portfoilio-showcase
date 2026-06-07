# Architecture

> A sanitized description of the architecture of a commercial Next.js/React SaaS
> web application. No proprietary code, client identifiers, or business logic
> are reproduced here.

---

## 1. Architectural Goals

The architecture was driven by four goals common to long-lived commercial
products:

1. **Maintainability** — a new contributor should locate any capability by name.
2. **Testability & predictability** — clear boundaries so behavior can be reasoned
   about in isolation.
3. **Resilience** — graceful handling of auth expiry, network failure, and SSR
   constraints.
4. **Scalability of the codebase** — adding a feature should be additive, not
   invasive.

---

## 2. Layered Architecture

The system is organized into four cooperating layers. Dependencies always point
*downward* — UI depends on state and services, services depend on the transport
layer, and never the reverse.

```
Presentation  →  State (client + server)  →  Service Layer  →  HTTP/WS Transport
```

### 2.1 Presentation Layer

- Built with the **Next.js App Router**, using file-system routing.
- Route groups separate the user workspace, the administration console, and an
  elevated operator surface.
- Components are split into **feature components** (orchestration) and **UI
  primitives** (presentation), keeping rendering logic reusable and accessible.

### 2.2 State Management

A deliberate split between *client state* and *server state*:

- **Client/UI state — Redux Toolkit.** Global UI concerns (session, layout,
  modals, feature view-state) live in slices created with `createSlice`. A
  `combineReducers` root composes them.
- **Persistence — redux-persist.** Only an explicit **whitelist** of slices is
  persisted to local storage. Server-side rendering uses a **noop storage**
  fallback so persistence never breaks during SSR.
- **Serialization guards.** Non-serializable values (e.g. `File` objects in
  transient flows) are explicitly excluded from the serializability check rather
  than disabling it globally.
- **Server/cache state — TanStack Query.** Remote data is cached, deduplicated,
  and background-refreshed by Query, keeping it out of Redux and avoiding stale
  global state.

### 2.3 Service Layer

Each domain capability owns a folder under `services/<feature>/`:

- `api.ts` — exposes named, typed functions for the feature's remote operations.
- `reducer.ts` — the feature's Redux slice (where client state is needed).

This is the codebase's unit of modularity: one feature → one service + one slice.
UI components never construct requests directly; they call a feature's `api.ts`.

### 2.4 HTTP / Transport Integration

- A small set of **typed request helpers** (`get`, `post`, `patch`, `put`, `del`,
  multipart, and streaming variants) wraps all network access.
- Two **Axios clients** are configured: a *public* client for unauthenticated
  calls and an *authenticated* client that attaches the bearer token via a
  request interceptor.
- A **response interceptor** centralizes authentication failure handling:
  - detects both HTTP `401` and in-payload auth-failure signals,
  - performs a **single-flight token refresh** (concurrent failures share one
    refresh request),
  - retries the original request transparently, and
  - tears down the session and redirects to login if refresh is impossible.
- **Real-time transport** is provided by a configured `socket.io-client`
  connection plus a streaming `fetch` wrapper for incremental responses, with the
  same refresh-aware resilience.

---

## 3. Cross-Cutting Concerns

| Concern | Approach |
|---------|----------|
| **Routing & access control** | Next.js middleware inspects the session cookie, decodes the role claim, and redirects unauthorized access *before* the page renders. |
| **Validation** | Zod schemas define input contracts; types are inferred from schemas (single source of truth). Locale-aware schema variants support i18n error messages. |
| **Internationalization** | i18next/react-i18next with per-locale message catalogs and right-to-left support. |
| **Theming** | `next-themes` with Tailwind design tokens. |
| **Configuration** | Environment values injected at build/run time from segregated env files; no secrets in source. |
| **Error feedback** | Centralized toast notifications for user-facing errors; console diagnostics kept free of sensitive payloads. |

---

## 4. Utility Organization

Shared, framework-agnostic logic lives under `utils/`, partitioned by intent:

```
utils/
├── constants/     ← static config values
├── enums/         ← endpoints, HTTP/auth status, roles, domain enums
├── helpers/       ← pure functions, request helpers, formatters, exporters
├── integration/   ← configured Axios clients & interceptors
├── interfaces/    ← shared TypeScript types
├── lib/           ← i18n setup, parsers
└── validations/   ← Zod schemas (incl. i18n variants)
```

This keeps business-agnostic concerns reusable and prevents feature code from
re-implementing primitives.

---

## 5. Why This Architecture

- **Feature ownership** reduces merge conflicts and makes responsibilities clear
  in code review.
- **The service/transport split** means cross-cutting changes (auth, retries,
  headers) happen in *one* place.
- **Separating client and server state** avoids the common anti-pattern of
  mirroring server data into Redux and manually keeping it in sync.
- **Strict typing + schema-derived types** removes a whole category of
  integration bugs before runtime.

See [`engineering-practices.md`](engineering-practices.md) for the practices that
keep this architecture healthy over time, and [`../diagrams/`](../diagrams/) for
visual representations.
