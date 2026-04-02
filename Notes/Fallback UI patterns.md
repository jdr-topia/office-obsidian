---
created: 2026-04-02
tags: [error-handling, ux, resilience]
related: [ [[Error handling MOC]], [[Error boundaries in React]], [[API error handling]] ]
status: active
---

# Fallback UI patterns

Fallback UIs are what the user sees when the application cannot display the expected content. Good fallbacks maintain user trust and reduce frustration.

## The Spectrum of Failure States

A well-designed UI should explicitly handle every loading/error state:

| State | Description | Pattern |
|---|---|---|
| **Loading** | Data is being fetched | Skeleton Screen / Spinner |
| **Empty** | No data exists yet | Empty State Illustration |
| **Error** | Request failed | Error State with CTA |
| **Offline** | No network connection | Offline Banner |
| **Not Found** | Resource doesn't exist | 404 Page |
| **Forbidden** | Access denied | 403 Page |

---

## Pattern 1: Skeleton Screens

Display a placeholder that mimics the shape of the content while it loads. This is significantly better than spinners because it reduces perceived load time.

```tsx
function UserCardSkeleton() {
  return (
    <div className="card skeleton">
      <div className="skeleton-avatar" />
      <div className="skeleton-line short" />
      <div className="skeleton-line long" />
    </div>
  );
}

function UserCard({ userId }) {
  const { data, isLoading, isError } = useUser(userId);

  if (isLoading) return <UserCardSkeleton />;
  if (isError) return <ErrorState message="Could not load user." />;
  return <UserCardContent user={data} />;
}
```

---

## Pattern 2: Error State with CTA (Call to Action)

The error state should not be a dead end. Always provide the user with a next step.

```tsx
function ErrorState({ message, onRetry }) {
  return (
    <div className="error-state">
      <WarningIcon />
      <h2>Something went wrong</h2>
      <p>{message}</p>
      {onRetry && <button onClick={onRetry}>Try Again</button>}
    </div>
  );
}
```

---

## Pattern 3: Next.js `error.tsx` and `not-found.tsx`

Next.js App Router has built-in file conventions for error states at the route level.

```tsx
// app/dashboard/error.tsx
"use client";

export default function DashboardError({ error, reset }) {
  return (
    <div>
      <h2>Failed to load dashboard</h2>
      <p>{error.message}</p>
      <button onClick={reset}>Try again</button>
    </div>
  );
}
```

```tsx
// app/users/[id]/not-found.tsx
export default function UserNotFound() {
  return <div>This user does not exist.</div>;
}
```

---

## Pattern 4: Progressive Disclosure

For complex pages, load the most critical content first and show degraded state only for the parts that failed. Use [[Error boundaries in React]] at the widget level.

---

## Related Topics
- [[Error handling MOC]]
- [[Error boundaries in React]]
- [[API error handling]]
- [[Performance optimization MOC]] (Skeleton screens improve perceived performance)
