# Security & Privacy Considerations

> Security and data-protection practices applied during delivery of a commercial
> SaaS web application. This document describes *approaches*, not implementation
> details. No secrets, endpoints, tokens, or client data appear here.

---

## 1. Secure Configuration Management

- Application configuration is **environment-scoped** (separate staging and
  production configuration), injected at build/run time.
- Only **public, client-safe** values are exposed to the browser bundle (via
  `NEXT_PUBLIC_*` variables): the API base URL, the payment provider's
  *publishable* key, and the socket URL.
- **No secrets are hard-coded** in application source. Deployment-time secrets are
  supplied through the CI provider's **encrypted secret store** and surfaced as
  environment variables to the pipeline only.

---

## 2. Authentication & Session Handling

The authentication design follows defense-in-depth principles:

- **Short-lived access tokens** paired with **refresh tokens**.
- A **single-flight refresh** mechanism: when multiple requests fail with an
  expired token simultaneously, only one refresh request is issued and all
  pending requests reuse its result — avoiding refresh storms and race
  conditions.
- **Transparent retry**: once refreshed, the original request is retried
  automatically with the new credentials.
- **Fail-closed teardown**: if refresh is not possible, the session is cleared
  (tokens and auth cookie removed) and the user is redirected to authentication.
- **Robust failure detection**: both HTTP `401` responses and APIs that embed
  authentication failures inside an otherwise-`200` payload are handled.
- **Multi-factor authentication** and **trusted-device** flows are part of the
  authentication surface.

---

## 3. Principle of Least Privilege (Authorization)

- **Role-based access control** is enforced at the edge using Next.js
  **middleware**, *before* protected pages render.
- The middleware decodes the role claim from the session and:
  - redirects unauthenticated users away from protected areas,
  - restricts the administration console to the admin role,
  - restricts the elevated operator surface to explicitly permitted roles.
- Authorization decisions are centralized, not scattered across individual pages.

---

## 4. Sensitive-Data & Logging Hygiene

- Diagnostic logging is kept **free of sensitive payloads** (tokens, credentials,
  personal data).
- User-facing error feedback is delivered through a centralized notification
  layer, decoupled from raw error internals.
- Client-side persistence is **explicitly whitelisted** — only the slices that
  must survive a reload are persisted; transient/sensitive data is not.

---

## 5. GDPR Awareness

For a product serving the European market, data protection is a design input,
not an afterthought:

- **Data minimization** — only the state required for UX is persisted client-side.
- **Purpose limitation** — configuration and feature flags are environment-scoped.
- **Transparency & control** — account and preference management are first-class
  surfaces in the product.

> Note: GDPR compliance is an organizational responsibility spanning legal,
> backend, and data layers. This document reflects the **front-end engineering
> practices** that support it, not a compliance certification.

---

## 6. Dependency & Supply-Chain Hygiene

- **Pinned dependencies** with a committed lockfile for reproducible installs.
- **`engines` constraints** ensure builds run on a known, supported runtime.
- Reliance on **well-maintained, widely-audited libraries** (Radix UI, TanStack,
  Redux Toolkit, Zod, Axios) over bespoke security-sensitive code.

---

## 7. Threats Explicitly Considered

| Threat | Mitigation approach |
|--------|--------------------|
| Token theft via long-lived credentials | Short-lived access tokens + rotation |
| Refresh race conditions | Single-flight refresh |
| Unauthorized route access | Edge middleware RBAC, fail-closed |
| Secret leakage in source | CI secret store + env injection |
| Sensitive data in logs | Logging hygiene, no payload dumps |
| Stale/over-persisted state | Whitelisted persistence |

---

## Disclaimer

This document describes engineering practices observed in a delivered product. It
intentionally omits implementation specifics, endpoints, schemas, and any
information that could compromise the confidentiality or security of the original
system.
