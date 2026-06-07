# Reusable Component & Code Patterns

> **All code below is original, generic illustration** written specifically for
> this portfolio. It demonstrates the *patterns and conventions* used on the
> commercial project without reproducing any proprietary implementation, business
> logic, endpoints, or types.

---

## 1. Typed Redux Hooks (single source of truth for store access)

Instead of importing `useSelector`/`useDispatch` everywhere and re-annotating
types, wrap them once:

```ts
// hooks/redux.ts
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from '@/store';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;
```

**Pattern:** every component gets fully-typed state access for free; the store's
shape is defined in exactly one place.

---

## 2. A Thin, Typed Request Layer over a Configured Client

UI and services never speak to `axios` directly — they go through small,
intention-revealing helpers:

```ts
// utils/helpers/request.ts (illustrative)
import { publicClient, authClient } from '@/utils/integration/http';

async function unwrap<T>(p: Promise<{ data: T }>): Promise<T> {
  try {
    const res = await p;
    return res.data;
  } catch (err: any) {
    // Normalize: surface the server payload if present, else the message.
    throw err?.response?.data ?? err?.message;
  }
}

export const get = <T>(url: string, params?: object, isAuth = false) =>
  unwrap<T>((isAuth ? authClient : publicClient).get(url, { params }));

export const post = <T>(url: string, body: object, isAuth = false) =>
  unwrap<T>((isAuth ? authClient : publicClient).post(url, body));
```

**Pattern:** one place to normalize errors, choose the right client, and keep
call sites declarative.

---

## 3. Dual HTTP Clients with Resilient Token Refresh

A public client for unauthenticated calls and an authenticated client that
attaches credentials and recovers from token expiry — with **single-flight**
refresh so concurrent failures don't stampede the refresh endpoint:

```ts
// utils/integration/http.ts (illustrative)
import axios from 'axios';
import { getAccessToken, refreshSession, endSession } from '@/auth/session';

const baseURL = process.env.NEXT_PUBLIC_API_BASE_URL;

export const publicClient = axios.create({ baseURL });
export const authClient = axios.create({ baseURL });

authClient.interceptors.request.use((config) => {
  const token = getAccessToken();
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

let inFlightRefresh: Promise<string> | null = null;

authClient.interceptors.response.use(
  (res) => res,
  async (error) => {
    const original = error.config;
    const isAuthError = error.response?.status === 401;

    if (!isAuthError || original._retry) return Promise.reject(error);
    original._retry = true;

    try {
      // Share one refresh across all concurrent 401s.
      inFlightRefresh ??= refreshSession();
      const newToken = await inFlightRefresh;
      original.headers.Authorization = `Bearer ${newToken}`;
      return authClient(original);
    } catch (e) {
      endSession();             // fail closed: clear + redirect
      return Promise.reject(e);
    } finally {
      inFlightRefresh = null;
    }
  },
);
```

**Pattern:** authentication resilience lives in *one* interceptor; features stay
oblivious to token mechanics.

---

## 4. Schema-Derived Types with Zod (validator == type)

Define the contract once; infer the type from it so they can never drift:

```ts
// validations/profile.ts (illustrative)
import { z } from 'zod';

export const profileSchema = z.object({
  fullName: z.string().min(1, 'Full name is required'),
  email: z.string().email('Invalid email address')
    .transform((v) => v.toLowerCase()),
  password: z.string()
    .min(8, 'At least 8 characters')
    .regex(/(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/, 'Include upper, lower, and a number'),
});

export type ProfileInput = z.infer<typeof profileSchema>;
```

Wired into a form with React Hook Form:

```tsx
// (illustrative)
import { useForm } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { profileSchema, type ProfileInput } from '@/validations/profile';

function ProfileForm() {
  const { register, handleSubmit, formState: { errors } } =
    useForm<ProfileInput>({ resolver: zodResolver(profileSchema) });

  const onSubmit = (data: ProfileInput) => {/* call service */};
  // render inputs + errors…
}
```

**Pattern:** a single schema powers validation *and* the TypeScript type — no
duplicated definitions.

---

## 5. Variant-Driven UI Primitive (CVA + tailwind-merge)

A reusable, accessible primitive with declarative variants:

```tsx
// components/ui/button.tsx (illustrative)
import { cva, type VariantProps } from 'class-variance-authority';
import { twMerge } from 'tailwind-merge';
import clsx from 'clsx';

const button = cva(
  'inline-flex items-center justify-center rounded-md font-medium transition',
  {
    variants: {
      intent: {
        primary: 'bg-primary text-white hover:bg-primary/90',
        ghost: 'bg-transparent hover:bg-muted',
      },
      size: { sm: 'h-8 px-3 text-sm', md: 'h-10 px-4' },
    },
    defaultVariants: { intent: 'primary', size: 'md' },
  },
);

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement>
  & VariantProps<typeof button>;

export function Button({ intent, size, className, ...props }: ButtonProps) {
  return <button className={twMerge(clsx(button({ intent, size }), className))} {...props} />;
}
```

**Pattern:** styling becomes a typed API; consumers pick `intent`/`size`, never
copy class strings.

---

## 6. A Focused, Reusable Hook

Small hooks encapsulate one cross-cutting behavior and are reused widely:

```ts
// hooks/use-debounce.ts (illustrative)
import { useEffect, useState } from 'react';

/** Returns a value that only updates after `delay` ms of quiet. */
export function useDebounce<T>(value: T, delay = 500): T {
  const [debounced, setDebounced] = useState(value);

  useEffect(() => {
    const id = setTimeout(() => setDebounced(value), delay);
    return () => clearTimeout(id);
  }, [value, delay]);

  return debounced;
}
```

**Pattern:** generic, well-documented, side-effect-clean hooks — composable
across features (search inputs, autosave, etc.).

---

## 7. The Feature Module Convention

Every feature follows the same shape, so the codebase is predictable:

```
services/<feature>/
├── api.ts        # export const DoThingApi = (data) => post('/…', data, true)
└── reducer.ts    # createSlice({ name, initialState, reducers })
```

```ts
// services/<feature>/reducer.ts (illustrative)
import { createSlice } from '@reduxjs/toolkit';

const slice = createSlice({
  name: 'feature',
  initialState: { selectedId: null as string | null },
  reducers: {
    setSelectedId(state, action: { payload: string | null }) {
      state.selectedId = action.payload;
    },
  },
});

export const { setSelectedId } = slice.actions;
export default slice.reducer;
```

**Pattern:** one feature → one `api.ts` + one slice. New capabilities are
*additive*, and reviewers always know where to look.

---

> Reminder: these snippets are deliberately generic teaching examples. They
> illustrate conventions and reasoning — not the proprietary product's code.
