---
created: 2026-04-10
tags: [scalability, monorepo, nx, turborepo]
related: [ [[Scalability MOC]], [[Module federation]], [[Design systems and shared libraries]] ]
status: active
---

# Monorepo architecture

A **monorepo** is a single repository that contains multiple distinct projects (apps, packages, services). It is the standard approach for large-scale frontend development at companies like Google, Meta, and Vercel.

## Monorepo vs Polyrepo

| | Monorepo | Polyrepo |
|---|---|---|
| **Dependency sharing** | Single `node_modules`, easy sharing | Duplicated dependencies, versioning hell |
| **Cross-project changes** | One PR changes everything atomically | Multiple PRs, coordination overhead |
| **Tooling** | Shared ESLint, TypeScript, Prettier configs | Each repo maintains its own tooling |
| **CI/CD** | Affected-only builds (fast) | Each repo has its own pipeline |
| **Discoverability** | All code in one place | Scattered across many repos |
| **Scale** | Requires tooling (Nx, Turborepo) | Easier to start but doesn't scale |

---

## Structure

```
my-monorepo/
├── apps/
│   ├── web/           ← Next.js customer app
│   ├── admin/         ← Next.js admin dashboard
│   └── mobile/        ← React Native app
├── packages/
│   ├── ui/            ← Shared component library
│   ├── utils/         ← Shared helper functions
│   ├── config/        ← Shared ESLint, TS, Prettier configs
│   └── api-client/    ← Shared typed API client
├── package.json       ← Root package.json (workspaces)
└── turbo.json         ← Turborepo pipeline config
```

---

## Tools: Turborepo vs Nx

### Turborepo (by Vercel)
The simpler, faster option for most teams. Focuses on **build task orchestration and caching**.

```json
// package.json (root)
{
  "workspaces": ["apps/*", "packages/*"]
}
```

```json
// turbo.json
{
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],  // Build packages before apps
      "outputs": [".next/**", "dist/**"]
    },
    "test": {
      "dependsOn": ["^build"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    }
  }
}
```

```bash
# Run build only for affected packages (with remote caching)
npx turbo build --filter=web...

# Run all dev servers in parallel
npx turbo dev
```

### Nx (by Nrwl)
More opinionated and feature-rich — includes code generators, project graph visualization, and first-class support for many frameworks.

```bash
# Create an Nx workspace
npx create-nx-workspace@latest
# Generate a Next.js app
nx generate @nx/next:app web
# Run only affected tests
nx affected --target=test
```

---

## Sharing Packages

```ts
// packages/utils/src/format.ts
export function formatCurrency(value: number, currency = 'USD'): string {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(value);
}
```

```json
// packages/utils/package.json
{ "name": "@myorg/utils", "main": "./src/index.ts" }
```

```ts
// apps/web/src/components/Price.tsx
import { formatCurrency } from '@myorg/utils';
// TypeScript resolves directly to source — no build step needed for local dev
```

---

## Related Topics
- [[Scalability MOC]]
- [[Module federation]]
- [[Design systems and shared libraries]]
- [[CI/CD for frontend]]
