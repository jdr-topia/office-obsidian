---
created: 2026-04-02
tags: [authentication, authorization, rbac]
related: [ [[Authentication and authorization MOC]], [[Protected routes]] ]
status: active
---

# RBAC in frontend

Role-Based Access Control (RBAC) is an authorization strategy where what a user can see or do is determined by their **role** (e.g., `admin`, `editor`, `viewer`).

## Why At the Frontend Level?

The backend is the true security boundary — it must always enforce RBAC on its APIs. However, frontend RBAC serves crucial UX goals:
- **Show/Hide Features**: An `admin` sees a "Delete" button; a `viewer` doesn't.
- **Route Guarding**: A `viewer` attempting to navigate to `/admin` is redirected.
- **Reduced Friction**: Users only see what they can actually use.

---

## Implementation Approach

### 1. Store Roles in the Auth State
After login, the user's role(s) should be part of the session or token payload.

```ts
// JWT payload example
{ "userId": "123", "role": "admin", "permissions": ["posts:write", "users:delete"] }
```

### 2. Create a Permission Hook or Utility

```ts
// hooks/usePermission.ts
import { useSession } from 'next-auth/react';

type Role = 'admin' | 'editor' | 'viewer';

export function usePermission(requiredRole: Role) {
  const { data: session } = useSession();
  const userRole = session?.user?.role as Role;

  const hierarchy: Record<Role, number> = { admin: 3, editor: 2, viewer: 1 };
  return hierarchy[userRole] >= hierarchy[requiredRole];
}
```

### 3. Conditionally Render UI

```tsx
function PostActions({ post }) {
  const canEdit = usePermission('editor');
  const canDelete = usePermission('admin');

  return (
    <div>
      {canEdit && <EditButton post={post} />}
      {canDelete && <DeleteButton post={post} />}
    </div>
  );
}
```

### 4. Role-Based Route Guards

```tsx
// Extend the ProtectedRoute component
const ProtectedRoute = ({ user, allowedRoles, redirectPath = '/403' }) => {
  if (!user || !allowedRoles.includes(user.role)) {
    return <Navigate to={redirectPath} replace />;
  }
  return <Outlet />;
};
```

---

## Coarse vs Fine-Grained Permissions

| Pattern | Example | Use Case |
|---|---|---|
| **Role-based** | `admin`, `editor`, `viewer` | Simple applications |
| **Permission-based** | `posts:write`, `users:delete` | Complex SaaS platforms |
| **Attribute-based (ABAC)** | "Can edit if own resource" | Dynamic, context-aware access |

---

## Related Topics
- [[Authentication and authorization MOC]]
- [[Protected routes]]
- [[Frontend security MOC]]
