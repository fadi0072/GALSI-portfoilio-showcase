# Architecture Overview (Diagrams)

> Mermaid diagrams describing the architecture at a high level. Sanitized — no
> proprietary modules, endpoints, or business logic are shown.

---

## System Layers

```mermaid
flowchart TD
    subgraph Presentation["Presentation Layer (Next.js App Router)"]
        R[Routes / Pages]
        FC[Feature Components]
        UI[Shared UI Primitives]
    end

    subgraph State["State Management"]
        RTK[Redux Toolkit + persist]
        RQ[TanStack Query]
    end

    subgraph Services["Service Layer (per-feature)"]
        API[api.ts]
        RED[reducer.ts slice]
    end

    subgraph Transport["HTTP / WS Integration"]
        HELP[Typed request helpers]
        AX[Axios clients + interceptors]
        WS[socket.io / streaming fetch]
    end

    EXT[(External APIs / Realtime)]

    R --> FC
    FC --> UI
    FC --> RTK
    FC --> RQ
    RQ --> API
    FC --> API
    API --> HELP
    RED --> RTK
    HELP --> AX
    AX --> EXT
    WS --> EXT
```

---

## Authentication & Token Refresh Flow

```mermaid
sequenceDiagram
    participant UI as UI / Feature
    participant SVC as Service (api.ts)
    participant AX as Authenticated Axios Client
    participant API as Backend API

    UI->>SVC: call feature operation
    SVC->>AX: request (+ bearer token)
    AX->>API: HTTP request
    API-->>AX: 401 / auth-failure payload
    Note over AX: Single-flight refresh<br/>(shared across concurrent failures)
    AX->>API: refresh token request
    alt refresh succeeds
        API-->>AX: new access + refresh tokens
        AX->>API: retry original request
        API-->>AX: 200 OK
        AX-->>SVC: data
        SVC-->>UI: result
    else refresh fails
        AX->>AX: clear session + cookie
        AX-->>UI: redirect to /auth
    end
```

---

## Route Protection (Edge Middleware)

```mermaid
flowchart TD
    REQ[Incoming request] --> MW{Middleware}
    MW -->|no session + protected route| AUTH[Redirect to /auth]
    MW -->|admin route + role != admin| HOME[Redirect to user home]
    MW -->|operator route + role not allowed| HOME
    MW -->|authorized| NEXT[Render page]
```

> Diagrams are intentionally generic: route names, role names, and module names
> shown here are illustrative placeholders, not the client's actual taxonomy.
