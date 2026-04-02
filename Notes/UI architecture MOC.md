---
created: 2026-04-02
tags: [moc, ui-architecture]
related: [ [[Frontend system design MOC]] ]
status: active
---

# UI architecture MOC

UI architecture defines how we structure, compose, and manage the lifecycle of our user interface components. A strong UI architecture ensures that components are reusable, testable, and scalable across large teams.

## Core Concepts & Patterns

### Component Organization
- [[Atomic design pattern]]: A methodology for creating design systems (Atoms -> Pages).
- [[Container vs presentational components]]: Separating logic from visual representation.

### Composition Patterns
- [[Headless components]]: Components that provide logic/state without any UI (e.g., Radix UI, TanStack Table).
- [[Compound components]]: A pattern for building complex components with shared state (e.g., Select, Tabs).
- **Higher-Order Components (HOC)**: Functional composition (mostly legacy in favor of Hooks).
- **Render Props**: Passing a function as a child (mostly legacy in favor of Hooks).

### Styling & Theming
- [[Design systems and shared libraries]]
- **CSS-in-JS vs CSS Modules vs Utility CSS**
- **Theming and Design Tokens**

---
## Related Topics
- [[Frontend system design MOC]]
- [[Design systems MOC]]
- [[Performance optimization MOC]] (Virtualization, Memoization)
