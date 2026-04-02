---
created: 2026-04-02
tags: [routing, spa, structure]
related: [ [[Routing strategy MOC]], [[Client side routing]] ]
status: active
---

# Nested and dynamic routes

Routing mechanisms that allow for more complex URL structures and flexible data-driven layouts.

## 1. Nested Routes (Layout Routing)
Building a UI where parts of the view change based on sub-routes while the parent layout remains fixed.

### The Problem
Traditional routing often re-renders the *entire* page. If you have a sidebar that appears on all `/settings/*` pages, you don't want to re-render it every time the user moves from `/settings/profile` to `/settings/appearance`.

### The Solution: `<Outlet />` (React Router) / `layout.js` (Next.js)
Each child route is "plugged into" a designated area of the parent route.

```tsx
// React Router:
<Route path="settings" element={<SettingsLayout />}>
  <Route index element={<ProfileSettings />} />
  <Route path="appearance" element={<AppearanceSettings />} />
</Route>
```

---

## 2. Dynamic Routes (Parameterized Routing)
Routes that contain variable segments, typically an ID, slug, or username.

### Standard Format
- `/users/:id`
- `/post/:slug`

### Parsing Parameters
- **React Router**: `useParams()` hook.
- **Next.js (App Router)**: `params` object passed directly into the page component.

```tsx
// Example: /users/123
function UserProfile({ params }) {
  const { id } = params;
  return <h1>Showing profile for User ID: {id}</h1>;
}
```

---

## Best Practices

1.  **Semantic URLs**: Use descriptive slugs (`/posts/how-to-react`) instead of IDs (`/posts/456`) for better SEO.
2.  **Fallback (Not-Found)**: Ensure a catch-all route (`*`) handles situations where no dynamic segments match.
3.  **Breadcrumbs**: Use current route segments to generate a navigation path (e.g., `Home > Settings > Profile`).

---
## Related Topics
- [[Client side routing]]
- [[Routing strategy MOC]]
- [[Rendering strategies MOC]]
