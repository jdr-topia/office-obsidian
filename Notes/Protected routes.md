---
created: 2026-04-02
tags: [routing, security, authentication]
related: [ [[Routing strategy MOC]], [[Authentication and authorization MOC]] ]
status: active
---

# Protected routes

Protected routes are routes that are only accessible if the user meets specific criteria, most commonly being authenticated (logged in) or having a certain role (RBAC).

## Implementation Pattern (React)

Create a "Wrapper Component" that checks the current user's session state.

```tsx
import { Navigate, Outlet } from 'react-router-dom';

const ProtectedRoute = ({ user, redirectPath = '/login' }) => {
  if (!user) {
    // Redirect to login if user is not authenticated
    return <Navigate to={redirectPath} replace />;
  }
  // User is authenticated, render the child routes
  return <Outlet />;
};

// Application shell:
<Routes>
  <Route index element={<LandingPage />} />
  <Route path="login" element={<LoginPage />} />
  <Route element={<ProtectedRoute user={currentUser} />}>
    <Route path="dashboard" element={<Dashboard />} />
    <Route path="settings" element={<Settings />} />
  </Route>
</Routes>
```

---

## Edge Cases & Optimization

### 1. The "Loading State" Problem
When a user visits a protected route, the application needs time to verify the authentication token. During this time, the `user` state might be `null`.
- **Solution**: Show a global loading spinner or a skeleton UI until the session is verified.

### 2. Client Side vs Server Side Protection (Next.js)
- **Client Side**: Check authentication in a hook/useEffect. (Less secure, UI might flash).
- **Server Side (Middleware)**: Use Next.js Middleware (`middleware.ts`) to intercept the request *before* the page is even rendered. Redirect at the edge.

### 3. Role Based Access (RBAC)
Pass authorized roles as props to your ProtectedRoute component.

```tsx
<Route element={<ProtectedRoute user={user} allowedRoles={['admin']} />}>
  <Route path="admin-panel" element={<AdminPanel />} />
</Route>
```

---

## Why Is This Important?

1.  **UX**: Prevents unauthorized users from seeing dashboards and private content.
2.  **Performance**: Avoids booting up large authenticated-only chunks for unauthenticated users.
3.  **Basic Security**: While client-side checks can be bypassed by a determined user, it's the first line of defense before API/Server security kicks in.

---
## Related Topics
- [[Authentication and authorization MOC]]
- [[Frontend security MOC]]
- [[Routing strategy MOC]]
