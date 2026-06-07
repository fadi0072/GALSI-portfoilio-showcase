# Engineering Portfolio — Commercial SaaS Web Application

> A sanitized, NDA-safe showcase of the engineering practices, architecture, and
> delivery process I applied while building and shipping a production commercial
> web application. **No proprietary source code, client identifiers, business
> logic, or confidential assets are included.**

---

## Project Overview

This repository documents my engineering contribution to a **commercial,
production-grade web application developed and delivered for a client in a
real production environment**.

The product is a multi-module **SaaS platform** built as a single-page-style
web application on top of a modern React/Next.js stack, featuring:

- A role-segmented experience (end-user workspace, administration console, and
  an elevated operator role) gated by server-side route protection.
- Real-time, streaming-driven interactions backed by WebSocket and
  server-sent-style streaming transports.
- Subscription and billing flows integrated with a third-party payment provider.
- Document processing and multi-format export (PDF, DOCX, PPTX, XLSX).
- Full internationalization (multiple languages, including right-to-left support).

> Product names, client identity, domain-specific terminology, business rules,
> API contracts, and proprietary algorithms have been **intentionally omitted**.
> What remains is a faithful description of *how* the software was engineered.

---

## My Responsibilities

On this engagement my responsibilities spanned the full delivery lifecycle:

| Area | Contribution |
|------|--------------|
| **Feature development** | Implemented end-to-end features across user, admin, and operator surfaces. |
| **Architecture decisions** | Defined the feature-based service layer, state-management boundaries, and shared-component strategy. |
| **API integration** | Built a typed HTTP/service layer with a centralized client, interceptors, and resilient token-refresh handling. |
| **Real-time integration** | Integrated WebSocket and streaming transports for live, incremental UI updates. |
| **Collaboration** | Worked within a Git-based pull-request workflow with code review and a protected `main` branch. |
| **Performance improvements** | Optimized re-render behavior, server-state caching, debouncing, and build output. |
| **Bug fixing** | Diagnosed and resolved production and pre-production defects across the stack. |
| **Release preparation** | Maintained environment segregation (staging/production) and CI/CD-driven deployments. |
| **Production delivery** | Shipped to a live environment via an automated deployment pipeline. |

---

## Technology Stack

> Only technologies actually present in the delivered codebase are listed.

| Layer | Technologies | Purpose |
|-------|--------------|---------|
| **Framework** | Next.js 15 (App Router), React 19 | SSR/CSR hybrid rendering, routing, and component model |
| **Language** | TypeScript 5 (`strict` mode) | End-to-end static type safety |
| **Client state** | Redux Toolkit, React-Redux, redux-persist | Predictable global state with selective persistence |
| **Server state** | TanStack Query (+ ESLint plugin, devtools) | Caching, background refetching, request deduplication |
| **Forms & validation** | React Hook Form, Zod, `@hookform/resolvers` | Type-safe, schema-driven form handling |
| **UI system** | Tailwind CSS 4, Radix UI primitives, `class-variance-authority`, `clsx`, `tailwind-merge` | Accessible, composable, design-token-driven UI |
| **Icons & theming** | `lucide-react`, `react-icons`, `next-themes` | Iconography and light/dark theming |
| **HTTP layer** | Axios (dual public/authenticated clients + interceptors) | Centralized, resilient API access |
| **Real-time** | `socket.io-client`, streaming fetch | Live and incremental data delivery |
| **Internationalization** | i18next, react-i18next, next-i18next | Multi-language + RTL support |
| **Payments** | Stripe (`@stripe/react-stripe-js`, `@stripe/stripe-js`) | Subscription & billing integration |
| **Data & tables** | TanStack Table, Recharts, Mermaid | Tabular data, charts, and diagrams |
| **Documents & export** | jsPDF, docx, pptxgenjs, xlsx-js-style, file-saver, react-pdf-viewer, react-dropzone | File upload, preview, and multi-format export |
| **Tooling** | ESLint, Prettier, `dotenv-cli`, Turbopack, rimraf | Linting, formatting, env management, fast dev builds |
| **CI/CD & hosting** | GitHub Actions → Azure Static Web Apps | Automated build and deployment |

---

## Architecture Overview

The application follows a **feature-based, layered architecture** with strict
separation between presentation, state, and data access. A high-level view:

```
┌─────────────────────────────────────────────────────────────┐
│                     Presentation Layer                        │
│   App Router routes  ·  Feature components  ·  UI primitives  │
└───────────────┬───────────────────────────────┬──────────────┘
                │                               │
        ┌───────▼────────┐              ┌───────▼─────────┐
        │  Client State  │              │   Server State  │
        │  Redux Toolkit │              │  TanStack Query │
        │ (+persist)     │              │                 │
        └───────┬────────┘              └───────┬─────────┘
                │                               │
        ┌───────▼───────────────────────────────▼─────────┐
        │                Service Layer                     │
        │     services/<feature>/api.ts + reducer.ts       │
        └───────────────────────┬──────────────────────────┘
                                │
        ┌───────────────────────▼──────────────────────────┐
        │      HTTP Integration (typed fetch helpers)       │
        │   Axios clients · interceptors · token refresh    │
        └───────────────────────┬──────────────────────────┘
                                │
                         External APIs / WS
```

Key principles:

- **Feature-based organization** — each domain capability owns its API calls and
  state slice under `services/<feature>/`.
- **Shared component strategy** — generic, accessible primitives live in a single
  `components/ui/` library, composed by feature components.
- **Service layer separation** — UI never calls HTTP directly; it goes through a
  feature service, which goes through typed request helpers, which use a single
  configured Axios layer.
- **State management split** — client/UI state in Redux Toolkit; server/cache
  state in TanStack Query; persistence is explicit and whitelisted.
- **Utility layers** — `enums`, `interfaces`, `helpers`, `validations`, `lib`, and
  `constants` are cleanly separated under `utils/`.
- **Configuration management** — environment-specific config is injected at build
  time via segregated environment files; secrets never live in source.
- **Navigation organization** — file-system routing via the App Router, with
  middleware-based, role-aware access control.

A fuller, sanitized example tree is in
[`examples/folder-structure-example.md`](examples/folder-structure-example.md),
and diagrams are in [`diagrams/`](diagrams/).

---

## Engineering Practices

These practices align with the expectations of the German engineering market
(clarity, correctness, maintainability, and accountability). Each is backed by
evidence in the delivered codebase:

- **Clean architecture & separation of concerns** — layered boundaries between
  presentation, state, services, and transport.
- **SOLID-aligned design** — single-responsibility modules (one feature → one
  service + one slice), dependency inversion through a shared request layer.
- **Type safety** — TypeScript `strict` mode, Zod-inferred form types, typed
  Redux hooks (`useAppSelector` / `useAppDispatch`).
- **Reusable components** — a centralized primitive library with variant-driven
  styling (`class-variance-authority`).
- **Code review culture & PR workflow** — Git pull-request flow into a protected
  `main` branch, with CI gating deployment.
- **Documentation standards** — consistent module banners, JSDoc on shared hooks,
  and structured READMEs.
- **Dependency management** — pinned, audited dependencies; `engines` constraints
  on Node/npm versions.
- **Environment segregation** — distinct staging/production env files and build
  commands via `dotenv-cli`.
- **Defensive programming** — SSR-safe storage fallbacks, network-error handling,
  resilient token refresh, and serialization guards.

See [`docs/engineering-practices.md`](docs/engineering-practices.md) for detail.

---

## Quality Assurance

Quality was maintained through a combination of tooling and disciplined process:

- **Static analysis** — ESLint (Next.js core-web-vitals + TypeScript rules) and
  Prettier enforced as an error-level lint rule.
- **Type checking** — `strict` TypeScript as a first line of defense against
  whole classes of runtime errors.
- **Manual validation & regression** — feature-level verification across role
  surfaces before merge and release.
- **Release verification** — staging-first promotion, with production builds
  validated through the CI pipeline before deploy.

See [`docs/quality-assurance.md`](docs/quality-assurance.md).

---

## Security & Privacy

Security and data protection were treated as first-class concerns:

- **Secure configuration management** — secrets injected through the CI secret
  store and environment variables, never committed in plaintext to the source.
- **Secret handling** — payment publishable keys and API base URLs are passed as
  build-time environment variables.
- **Authentication hardening** — short-lived access tokens with refresh-token
  rotation, single-flight refresh, and automatic session teardown on failure.
- **Principle of least privilege** — role-based route protection enforced in
  middleware; elevated surfaces require explicit role claims.
- **GDPR awareness** — privacy-conscious handling of user data and avoidance of
  sensitive data in logs.

See [`docs/security-considerations.md`](docs/security-considerations.md).

---

## Repository Map

```
portfolio-showcase/
├── README.md                         ← you are here
├── LICENSE
├── docs/
│   ├── architecture.md               ← layered architecture deep-dive
│   ├── engineering-practices.md      ← practices + evidence
│   ├── delivery-process.md           ← Git flow, CI/CD, environments
│   ├── security-considerations.md    ← auth, secrets, privacy
│   └── quality-assurance.md          ← QA, linting, release verification
├── diagrams/
│   ├── architecture-overview.md      ← system & layer diagrams (Mermaid)
│   ├── component-flow.md             ← request/data-flow diagrams
│   └── folder-structure.md           ← sanitized structure diagram
├── examples/
│   ├── folder-structure-example.md   ← illustrative (non-proprietary) tree
│   └── reusable-component-patterns.md← generic, original example patterns
└── assets/
    └── placeholders/                 ← placeholder assets only
```

---

## Professional Disclaimer

> This repository demonstrates engineering approaches, architectural decisions,
> and development practices applied during the delivery of a commercial product.
> Proprietary source code, client-specific logic, and confidential information
> have been intentionally excluded to respect contractual obligations and
> confidentiality agreements.

All code samples in this repository are **original, generic illustrations**
written for portfolio purposes. They do not reproduce the client's
implementation.

---

## License

Released under the [MIT License](LICENSE) — applies to the documentation and
illustrative examples in this repository only.
