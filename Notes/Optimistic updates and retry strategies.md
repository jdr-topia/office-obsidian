---
created: 2026-04-02
tags: [concept, user-experience]
related: [ [[Server state management]], [[Data fetching strategy MOC]], [[Frontend system design MOC]] ]
status: active
---

# Optimistic updates and retry strategies

## Overview

In modern web applications, user experience is paramount. Users expect the UI to feel **instant and resilient**, regardless of network latency or intermittent connectivity issues.

**Optimistic Updates** make the app feel faster by responding immediately to user input. **Retry Strategies** make the app feel stronger by automatically recovering from network failures.

Together, these patterns turn a "clunky" app into a "premium" experience.

---

## 1. Optimistic Updates

An optimistic update is a technique where the UI is updated immediately as if an action (like a POST or PATCH) succeeded, **before** the server response arrives.

If the request subsequently fails, the UI is **rolled back** to its previous state.

### Why Use It?
- **Perceived Performance**: Eliminates the delay between a user click and UI change.
- **Engagement**: Users can keep interacting with the UI without waiting for expensive requests to finish.

### Best Use Cases
| Feature | Best For? | Example |
| :--- | :--- | :--- |
| **Like / Heart** | ✅ Yes | Liking a post on social media |
| **Toggles** | ✅ Yes | Turning a notification setting ON |
| **Simple Edits** | ✅ Yes | Renaming a folder / changing a title |
| **Deleting Items** | ✅ Yes | Deleting an email (move to trash) |
| **Financial Trans.** | ❌ No | Withdrawing money (needs confirmation) |
| **Complex Logic** | ❌ No | Applying a rare discount code |

---

## 2. Retry Strategies

Network requests can fail for many reasons: temporary server overload, flaky Wi-Fi, or switching between cell towers. A retry strategy attempts to automatically recover from these errors.

### Common Retry Patterns

**A. Simple Retry**
Retry the request $N$ times immediately.
- **Problem**: Can overwhelm a struggling server (Thundering Herd Problem).

**B. Exponential Backoff**
Increase the delay between each retry attempt.
- **Example**: 1s, 2s, 4s, 8s ...
- **Benefit**: Gives the server time to recover.

**C. Jitter (Randomness)**
Adds a small random delay to prevent multiple clients from retrying at exactly the same time.

---

## Technical Implementation (Next.js & TanStack Query)

### Optimistic Update Example

```typescript
'use client'
import { useMutation, useQueryClient } from '@tanstack/react-query'

async function updateTodoStatus({ id, completed }) {
  const res = await fetch(`/api/todos/${id}`, {
    method: 'PATCH',
    body: JSON.stringify({ completed }),
  })
  if (!res.ok) throw new Error('Update failed')
  return res.json()
}

export function useOptimisticToggleTodo() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: updateTodoStatus,

    // Step 1: When mutate() is called, this runs immediately
    onMutate: async (newTodo) => {
      // Cancel outgoing refetches (so they don't overwrite our optimistic update)
      await queryClient.cancelQueries({ queryKey: ['todos'] })

      // Snapshot the previous state (for rollback)
      const previousTodos = queryClient.getQueryData(['todos'])

      // Step 2: Optimistically update the cache
      queryClient.setQueryData(['todos'], (old: any) =>
        old.map((t) => (t.id === newTodo.id ? { ...t, completed: newTodo.completed } : t))
      )

      // Step 3: Return context object with snapshot
      return { previousTodos }
    },

    // Step 4: If mutation fails, rollback using our snapshot
    onError: (err, newTodo, context) => {
      queryClient.setQueryData(['todos'], context?.previousTodos)
      // Notify user (e.g., Toast)
      alert('Failed to update todo. Rolling back.')
    },

    // Step 5: Always refetch after success/failure to ensure sync with server
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['todos'] })
    },
  })
}
```

### Retry Strategy Configuration

TanStack Query supports exponential backoff retries by default:

```typescript
// Query Client Setup
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 3, // Retry failed queries 3 times
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000), // Exponential backoff
    },
    mutations: {
       retry: 2, // Be careful with mutation retries (non-idempotent endpoints!)
    }
  },
})
```

---

## Key Considerations: Idempotency

Before retrying a **mutation** (like a POST to create an order), ensure the API endpoint is **idempotent**. An idempotent request can be repeated multiple times without changing the result beyond the initial call.

- **Idempotent**: `PUT /users/me` (always sets name to 'Raman').
- **NOT Idempotent**: `POST /orders/checkout` (creates a new order on every retry).

**Solution for POST retries**: Use an **Idempotency Key** in the header. The server checks if it has already processed a request with that key.

---

## Related Notes
- [[Server state management]]
- [[Data fetching strategy MOC]]
- [[Performance requirements]]
- [[Frontend system design MOC]]
