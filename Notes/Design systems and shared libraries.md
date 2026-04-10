---
created: 2026-04-10
tags: [scalability, design-systems, component-library, shared-packages]
related: [ [[Scalability MOC]], [[Design systems MOC]], [[Monorepo architecture]] ]
status: active
---

# Design systems and shared libraries

A **design system** is a collection of reusable UI components, design tokens, patterns, and guidelines that ensure consistent user experience across products and teams. A **shared library** is its technical implementation — an npm package that teams import.

## Why It Matters at Scale

Without a shared system:
- 5 teams build 5 versions of a `Button` component.
- Each has slightly different padding, hover states, and accessibility.
- A brand update requires changing 5 codebases.
- Users experience visual inconsistency across the product.

---

## The Anatomy of a Design System

```
Design System
├── Design Tokens  ← Color palette, spacing, typography, shadows (source of truth)
├── Components     ← Button, Modal, Input built from tokens
├── Patterns       ← How components compose (Form layout, Data table pattern)
├── Documentation  ← Storybook: live, interactive component docs
└── Governance     ← How to contribute new components, who owns the system
```

---

## Building a Shared Component Library (Monorepo)

### 1. Structure the Package

```
packages/ui/
├── src/
│   ├── Button/
│   │   ├── Button.tsx
│   │   ├── Button.test.tsx
│   │   └── Button.stories.tsx
│   ├── Input/
│   └── index.ts        ← Re-exports everything
├── package.json
└── tsconfig.json
```

### 2. Export Cleanly

```ts
// packages/ui/src/index.ts
export { Button } from './Button/Button';
export type { ButtonProps } from './Button/Button';
export { Input } from './Input/Input';
```

### 3. Build for Consumption

```json
// packages/ui/package.json
{
  "name": "@myorg/ui",
  "main": "./dist/index.js",
  "module": "./dist/index.mjs",
  "types": "./dist/index.d.ts",
  "exports": {
    ".": {
      "import": "./dist/index.mjs",
      "require": "./dist/index.js",
      "types": "./dist/index.d.ts"
    }
  }
}
```

---

## Versioning Strategy

| Strategy | How | Best For |
|---|---|---|
| **Synchronized versioning** | All packages share one version (e.g., `1.5.0`) | Monorepo, tight coupling |
| **Independent versioning** | Each package has its own semver | Published npm packages |
| **Lockstep (internal)** | Unversioned, always use workspace latest | Internal monorepo only |

---

## Governance: The Contribution Model

| Model | Who adds components | Best For |
|---|---|---|
| **Centralized** | Only the design system team | Consistency, slower velocity |
| **Federated** | Any team, reviewed by DS team | Faster, more risk |
| **Open** | Any team, any time | Small orgs with high trust |

---

## Related Topics
- [[Scalability MOC]]
- [[Design systems MOC]]
- [[Monorepo architecture]]
- [[Atomic design pattern]]
