---
created: 2026-04-10
tags: [moc, design-systems, component-library, storybook]
related: [ [[Frontend system design MOC]], [[Scalability MOC]], [[UI architecture MOC]] ]
status: active
---

# Design systems MOC

A design system is the single source of truth for a product's visual language and interaction patterns. It is the contract between design and engineering.

## Core Building Blocks

| Layer | What it is | Notes |
|---|---|---|
| **Design Tokens** | Named variables for all visual values | Colours, spacing, type, radii, shadows |
| **Component Library** | Reusable UI components built from tokens | Button, Input, Modal, Table |
| **Pattern Library** | Compositions of components for common tasks | Form layout, Data table, Navigation |
| **Documentation** | Interactive reference for designers + devs | Storybook |

## Notes in this Section
- [[Component libraries]]: Choosing between off-the-shelf and custom component libraries.
- [[Design tokens and theming]]: Using tokens as the single source of visual truth.
- [[Storybook]]: Developing, documenting, and testing components in isolation.

## Famous Design Systems

| System | Built by | Stack |
|---|---|---|
| Material UI (MUI) | Google | React |
| Ant Design | Alibaba | React |
| Chakra UI | Community | React |
| Radix UI | WorkOS | React (Headless) |
| Shadcn/ui | Community | React + Radix + Tailwind |
| Carbon | IBM | React |

---
## Related Topics
- [[Frontend system design MOC]]
- [[UI architecture MOC]]
- [[Scalability MOC]]
- [[Atomic design pattern]]
