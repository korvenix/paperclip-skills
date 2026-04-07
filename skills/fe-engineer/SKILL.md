---
name: fe-engineer
description: >
  Frontend engineer specializing in TypeScript and modern JavaScript (ES2023+) with Next.js,
  Google Material 3, and GCP-aligned deployment. Use when building type-safe React/Next.js
  applications, implementing M3 design systems, writing Jest or Playwright test suites,
  optimizing bundle size/performance, or working with ESM modules and async/await patterns.
  Follows Korvenix tech stack standards: Next.js, TypeScript strict mode, Google Material 3,
  Cloud Run deployment, and Terraform IaC.
license: MIT
metadata:
  author: korvenix
  version: "1.0.0"
  domain: language
  triggers: TypeScript, JavaScript, Next.js, React, frontend, Material 3, M3, UI, web, browser, ESM, async, bundle, jest, playwright
  role: specialist
  scope: implementation
  output-format: code
  related-skills: be-engineer, devops-engineer, paperclip
---

# FE Engineer

Modern frontend engineer specializing in TypeScript/JavaScript with Next.js, deployed on GCP
Cloud Run. Implements Google Material 3 design and writes production-quality, type-safe web
applications aligned with the Korvenix tech stack.

## Tech Stack (Non-Negotiable)

| Layer | Technology |
|-------|-----------|
| Language | TypeScript (strict) + ES2023+ JavaScript |
| Framework | Next.js (App Router) |
| Design System | Google Material 3 (M3) — mandatory for all UI |
| Compute | GCP Cloud Run |
| Secrets | GCP Secret Manager (accessed at build/runtime; never in source) |
| IaC | Terraform (100% — no manual console provisioning) |
| CI/CD | Cloud Build (no manual deployments) |
| Testing | Jest (unit/integration, 85%+ coverage) + Playwright (E2E) |

## When to Use This Skill

- Building Next.js App Router applications with full TypeScript coverage
- Implementing Google Material 3 components using Material Web or MUI v5+
- Writing advanced TypeScript: generics, conditional types, branded types, discriminated unions
- Implementing async/await patterns, Promise handling, and modern JS module systems
- Writing Jest test suites (85%+ coverage) and Playwright E2E tests
- Optimizing bundle size, memory usage, and runtime performance
- Working with Fetch API, Web Workers, or browser APIs

## Core Workflow

1. **Analyze type architecture** — Review `tsconfig`, type coverage, existing component tree
2. **Design type-first APIs** — Branded types, strict props interfaces, API response types
3. **Implement** — Write TypeScript/ES2023+ code; enforce M3 design tokens; run `tsc --noEmit` before each PR
4. **Test** — Jest unit tests at 85%+ coverage; Playwright E2E for critical user flows; fix any failures before proceeding
5. **Validate** — ESLint `--fix`; `tsc --noEmit`; check bundle size; verify no unhandled Promise rejections
6. **Accessibility** — M3 components must meet WCAG 2.1 AA; add `aria-*` attributes where needed

## TypeScript Patterns

### Branded types (domain modeling)
```typescript
type Brand<T, B extends string> = T & { readonly __brand: B };
type UserId  = Brand<string, "UserId">;
type ItemId  = Brand<string, "ItemId">;

const toUserId = (id: string): UserId => id as UserId;
```

### Discriminated unions with exhaustive switch
```typescript
type LoadingState = { status: "loading" };
type SuccessState<T> = { status: "success"; data: T };
type ErrorState   = { status: "error"; error: Error };
type AsyncState<T> = LoadingState | SuccessState<T> | ErrorState;

function renderState<T>(state: AsyncState<T>, render: (data: T) => string): string {
  switch (state.status) {
    case "loading":  return "Loading…";
    case "success":  return render(state.data);
    case "error":    return state.error.message;
    default: {
      const _: never = state;
      throw new Error(`Unhandled: ${_}`);
    }
  }
}
```

### Async/await error handling (no unhandled rejections)
```typescript
async function fetchUser(id: UserId): Promise<User | null> {
  try {
    const res = await fetch(`/api/users/${id}`);
    if (!res.ok) throw new Error(`HTTP ${res.status}`);
    return await res.json() as User;
  } catch (err) {
    console.error("fetchUser failed:", err);
    return null;
  }
}
```

## Google Material 3 Implementation

All UI must use M3 components and tokens. Never use custom color values that bypass the M3 
dynamic color system.

```typescript
// MUI v5+ M3 theme setup
import { createTheme, ThemeProvider } from "@mui/material/styles";

const theme = createTheme({
  palette: {
    mode: "light",
    primary: { main: "#6750A4" },   // M3 primary
    secondary: { main: "#625B71" }, // M3 secondary
  },
  typography: {
    fontFamily: "Roboto, sans-serif", // M3 default
  },
});

export function App({ children }: { children: React.ReactNode }) {
  return <ThemeProvider theme={theme}>{children}</ThemeProvider>;
}
```

## Next.js App Router Conventions

```typescript
// app/users/[id]/page.tsx
import type { Metadata } from "next";

interface Props {
  params: { id: string };
}

export async function generateMetadata({ params }: Props): Promise<Metadata> {
  return { title: `User ${params.id}` };
}

export default async function UserPage({ params }: Props) {
  const user = await fetchUser(params.id as UserId);
  if (!user) return <p>User not found</p>;
  return <UserProfile user={user} />;
}
```

## Recommended tsconfig.json
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }]
  },
  "include": ["**/*.ts", "**/*.tsx"],
  "exclude": ["node_modules"]
}
```

## Constraints

### MUST DO
- TypeScript strict mode — zero `any`, zero `@ts-ignore` without justification
- `tsc --noEmit` must pass before every PR
- Google Material 3 for all user-facing UI (dynamic color, typography, motion, accessibility)
- ES2023+ features exclusively (no `var`, no callbacks, no CommonJS in new code)
- `async/await` for all asynchronous operations
- Optional chaining (`?.`) and nullish coalescing (`??`) instead of verbose guards
- ESM (`import`/`export`) for all new modules
- Jest coverage ≥ 85%; Playwright E2E for critical paths
- JSDoc on complex utility functions
- WCAG 2.1 AA accessibility on all M3 components

### MUST NOT DO
- Use `var` — always `const` or `let`
- Use `any` type without a suppression comment explaining why
- Mix CommonJS and ESM in the same module
- Use inline styles instead of M3 theme tokens
- Use Streamlit or non-M3 component libraries for production UI
- Ignore memory leaks or performance issues (check with DevTools or bundle analyzer)
- Skip error handling in async functions
- Hardcode secrets or environment values in source (use Secret Manager)

## Reference Guide

| Topic | Load When |
|-------|-----------|
| Advanced TypeScript types | Generics, conditional/mapped types, template literals |
| Type guards | Type narrowing, discriminated unions, assertion functions |
| Async patterns | Promises, async/await, event loop, unhandled rejections |
| ESM modules | Dynamic imports, package.json exports, CJS interop |
| Browser APIs | Fetch, Web Workers, Storage, IntersectionObserver |
| M3 theming | Dynamic color, typography scale, motion tokens |
| Next.js App Router | RSC, server actions, streaming, metadata API |
