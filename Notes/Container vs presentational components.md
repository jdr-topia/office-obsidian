---
created: 2026-04-02
tags: [ui-architecture, react-patterns]
related: [ [[UI architecture MOC]], [[Client side rendering (CSR)]] ]
status: active
---

# Container vs presentational components

Also known as **"Smart vs Dumb"** components, this pattern focuses on the separation of concerns by splitting data fetching/logic from layout/visuals.

## Separation of Concerns

### 1. Presentational Components (Dumb)
They focus on how things look.
- **Data Source**: Receive data and callbacks purely via `props`.
- **Logic**: No state management (except UI state like a toggle) or side effects.
- **Dependency**: No dependencies on business logic, stores, or data fetchers.
- **Testing**: Highly unit testable due to pure functional nature.

### 2. Container Components (Smart)
They focus on how things work.
- **Data Source**: Connect to stores (Redux, TanStack Query, Context) and handle data fetching.
- **Logic**: Manage business logic and side effects (`useEffect`).
- **Dependency**: Highly coupled to data layers.
- **Testing**: Requires mocking/wrapping for testing.

---

## Evolution of the Pattern

With the introduction of **React Hooks**, this distinction has blurred but remains a critical architectural principle.

### Hook-Based Container (Modern)
Instead of a separate "Container" component, the **Custom Hook** often acts as the "smart" layer.

```tsx
// Use this pattern to keep your UI clean:
function UserProfileContainer({ userId }) {
  const { user, isLoading } = useUser(userId); // The smart logic
  
  if (isLoading) return <LoadingSpinner />;
  
  // The dumb view (Presentational)
  return <UserProfile user={user} onUpdate={handleUpdate} />;
}
```

---

## Why Use It?

1.  **Reusability**: Different containers can use the same presentational component.
2.  **Designer Friendly**: Styles are isolated from logic.
3.  **Performance**: Prevents re-renders across the entire tree if localized in a container.
4.  **Clarity**: At a glance, you know where logic and UI live.

---
## Related Patterns
- [[State management MOC]]
- [[UI architecture MOC]]
- [[Atomic design pattern]]
