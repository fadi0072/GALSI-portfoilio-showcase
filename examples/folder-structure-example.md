# Folder Structure Example (Sanitized)

> An **illustrative, generalized** representation of the project's organization.
> Real feature names and files are replaced with neutral placeholders. This is a
> demonstration of *organizational principles*, not a copy of the proprietary
> tree.

---

## Annotated Tree

```
project-root/
├── app/                          # Next.js App Router (file-system routing)
│   ├── layout.tsx                # Root layout (providers, theming)
│   ├── globals.css               # Global styles / Tailwind layer
│   ├── auth/                     # Authentication route group
│   ├── dashboard/                # End-user workspace (feature routes)
│   │   ├── layout.tsx            # Shared dashboard shell
│   │   ├── feature-a/            # One route folder per feature
│   │   │   ├── page.tsx          # List view
│   │   │   └── [id]/page.tsx     # Detail view (dynamic segment)
│   │   └── feature-b/
│   ├── admin/                    # Admin console route group (RBAC-gated)
│   └── api/                      # Route handlers (server-side endpoints)
│
├── components/
│   ├── ui/                       # Reusable primitives (button, dialog, table…)
│   ├── <feature>/                # Feature-specific composed components
│   └── popups/                   # Shared modal/drawer components
│
├── services/                     # *** The feature module unit ***
│   ├── <feature>/
│   │   ├── api.ts                # Typed remote operations for the feature
│   │   └── reducer.ts            # Redux slice for the feature's client state
│   └── socket/
│       ├── config.ts             # Real-time connection configuration
│       └── reducer.ts
│
├── store/
│   ├── index.ts                  # configureStore + persist setup
│   └── root-reducer.ts           # combineReducers of all feature slices
│
├── hooks/
│   ├── redux.ts                  # Typed useAppSelector / useAppDispatch
│   ├── use-debounce.ts           # Generic reusable hooks
│   └── use-<feature>-*.ts        # Feature-scoped behavior hooks
│
├── utils/
│   ├── constants/                # Static configuration values
│   ├── enums/                    # endpoint, http-status, roles, domain enums
│   ├── helpers/                  # Pure functions, request helpers, exporters
│   ├── integration/
│   │   └── axios.ts              # Configured clients + interceptors
│   ├── interfaces/               # Shared TypeScript types
│   ├── lib/                      # i18n init, parsers
│   └── validations/              # Zod schemas (+ i18n variants)
│
├── environments/                 # Per-environment configuration
│   ├── .env.staging
│   └── .env.production
│
├── public/
│   └── locales/                  # i18n message catalogs (per language)
│
├── .github/workflows/            # CI/CD pipeline definitions
├── middleware.ts                 # Edge route protection (RBAC)
├── next.config.ts                # Framework configuration
├── tsconfig.json                 # TypeScript (strict) + path aliases
├── .eslintrc.json                # Lint configuration
└── .prettierrc                   # Formatting configuration
```

---

## Why It's Organized This Way

| Decision | Rationale |
|----------|-----------|
| **`services/<feature>/{api,reducer}`** | Co-locates a feature's data access and state — find everything about a feature in one place. |
| **`components/ui` vs feature components** | Separates reusable presentation from feature orchestration. |
| **`utils/` partitioned by intent** | Keeps cross-cutting primitives discoverable and free of business coupling. |
| **`environments/` outside source** | Makes configuration explicit and environment-segregated. |
| **`middleware.ts` at the edge** | Centralizes authorization before render, not per-page. |
| **Path alias `@/*`** | Stable, refactor-friendly imports regardless of nesting depth. |

---

## Naming Conventions Observed

- **kebab-case** for files and folders (`use-debounce.ts`, `funding-navigator/`).
- **PascalCase** for React components and exported types.
- **camelCase** for functions, variables, and slice action creators.
- **Enums** centralize otherwise-magic strings (endpoints, statuses, roles).
- Feature folders read as **nouns**; hooks read as **verbs/behaviors**
  (`use-…`).
