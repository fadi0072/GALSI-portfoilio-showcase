# Component & Data Flow (Diagrams)

> Illustrative data-flow diagrams. Generic and non-proprietary.

---

## Read Path: Server State via TanStack Query

```mermaid
flowchart LR
    C[Feature Component] -->|useQuery| Q[TanStack Query cache]
    Q -->|cache miss / stale| SVC[Feature api.ts]
    SVC --> H[Request helper]
    H --> AX[Axios client]
    AX --> API[(Backend)]
    API --> AX --> H --> SVC --> Q
    Q -->|cached data| C
```

The component never touches the network directly; Query manages caching,
deduplication, and background refresh.

---

## Write Path: Forms → Validation → Service

```mermaid
flowchart LR
    F[Form Component] --> RHF[React Hook Form]
    RHF --> Z[Zod schema resolver]
    Z -->|valid| SVC[Feature api.ts]
    Z -->|invalid| ERR[Field-level errors in UI]
    SVC --> H[Request helper]
    H --> AX[Axios client]
    AX --> API[(Backend)]
    API --> TOAST[Centralized toast feedback]
```

Validation rules and the resulting TypeScript types come from a **single Zod
schema** — no duplication between the validator and the form's types.

---

## Client State Flow (Redux Toolkit)

```mermaid
flowchart LR
    UI[Component] -->|useAppDispatch| ACT[Slice action]
    ACT --> RED[Reducer]
    RED --> STORE[(Store)]
    STORE -->|persist whitelist| LS[(Local Storage)]
    STORE -->|useAppSelector| UI
```

Only whitelisted slices are persisted; SSR uses a noop storage fallback.

---

## Real-Time Stream Flow

```mermaid
sequenceDiagram
    participant UI as Component
    participant WS as socket.io / stream
    participant API as Backend
    UI->>WS: subscribe / open stream
    API-->>WS: incremental chunk
    WS-->>UI: onChunk → progressive render
    API-->>WS: completion event
    WS-->>UI: onComplete → finalize UI
```

> All entity names are placeholders for portfolio purposes.
