---
created: 2026-04-02
tags: [error-handling, api, network, tanstack-query]
related: [ [[Error handling MOC]], [[Server state management]], [[Optimistic updates and retry strategies]] ]
status: active
---

# API error handling

A robust strategy for handling every possible failure mode when communicating with a backend API.

## Categories of API Errors

| Status Code Range | Category | Handling Strategy |
|---|---|---|
| **4xx** | Client errors (bad request, unauthorized) | Show user-facing message, don't retry |
| **5xx** | Server errors (internal, gateway timeout) | Retry with backoff, show generic error |
| **Network Failure** | No response (offline, timeout) | Retry, show offline banner |

---

## Layer 1: Centralized API Client

Create a single Axios or Fetch wrapper that handles global error logic. This prevents duplicating error-handling logic across every component.

```ts
// lib/api.ts
import axios from 'axios';

const api = axios.create({ baseURL: '/api' });

api.interceptors.response.use(
  (response) => response,
  (error) => {
    const status = error.response?.status;

    if (status === 401) {
      // Token expired — redirect to login
      window.location.href = '/login';
    }

    if (status === 403) {
      // Forbidden — show access denied
      toast.error("You don't have permission to do that.");
    }

    if (status >= 500) {
      toast.error("Server error. Please try again later.");
    }

    return Promise.reject(error);
  }
);
```

---

## Layer 2: TanStack Query Error Handling

TanStack Query provides `isError`, `error`, and `onError` for granular, per-query control.

```tsx
const { data, isError, error } = useQuery({
  queryKey: ['user', id],
  queryFn: () => fetchUser(id),
  retry: 2,          // Retry failed requests up to 2 times
  retryDelay: 1000,  // Wait 1 second between retries
});

if (isError) {
  return <ErrorMessage message={error.message} />;
}
```

### Global Error Handling with QueryClient

```ts
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      onError: (error) => toast.error(`Something went wrong: ${error.message}`),
    },
  },
});
```

---

## Layer 3: User Communication Patterns

1. **Toast Notifications**: For transient, non-critical errors (e.g., "Failed to save changes").
2. **Inline Validation Messages**: For form field errors from the API (e.g., "Email already taken").
3. **Full-Page Error States**: For critical failures that prevent the page from loading (see [[Fallback UI patterns]]).
4. **Empty States**: Distinguish between "no data" and "error fetching data" — they need different UX.

---

## Related Topics
- [[Error handling MOC]]
- [[Optimistic updates and retry strategies]]
- [[Server state management]]
- [[Fallback UI patterns]]
