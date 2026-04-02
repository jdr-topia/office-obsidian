---
created: 2026-04-02
tags: [error-handling, react, resilience]
related: [ [[Error handling MOC]], [[Fallback UI patterns]] ]
status: active
---

# Error boundaries in React

An **Error Boundary** is a React component that catches JavaScript errors in its child component tree, logs them, and displays a fallback UI instead of crashing the entire application.

## The Problem

Without error boundaries, a JavaScript error in a component (like rendering `undefined.name`) would unmount the entire React tree and show a blank/broken screen.

## Class Component Implementation

Error boundaries must be class components because they rely on the `componentDidCatch` and `getDerivedStateFromError` lifecycle methods (no Hook equivalent yet).

```tsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    // Update state to show fallback UI on next render
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    // Log the error to a monitoring service (e.g., Sentry)
    console.error("Caught by ErrorBoundary:", error, info.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <h2>Something went wrong.</h2>;
    }
    return this.props.children;
  }
}
```

## Usage

Wrap any component or section of your app that could fail:

```tsx
<ErrorBoundary fallback={<ErrorPage />}>
  <UserDashboard />
</ErrorBoundary>
```

---

## Practical Patterns

### 1. Page-Level Boundary
Wrap each page/route to prevent a failure on one page from breaking navigation.

```tsx
// In your router/layout
<ErrorBoundary fallback={<GenericErrorPage />}>
  <Outlet />
</ErrorBoundary>
```

### 2. Widget-Level Boundary
Wrap isolated UI sections (charts, recommendations) so their failure doesn't break the main content.

### 3. `react-error-boundary` Library
A popular utility that provides a functional-component-friendly API and a `useErrorBoundary` hook for imperatively triggering the boundary.

```tsx
import { ErrorBoundary } from 'react-error-boundary';

<ErrorBoundary
  FallbackComponent={ErrorFallback}
  onError={(error, info) => logToSentry(error, info)}
  onReset={() => window.location.reload()}
>
  <MyWidget />
</ErrorBoundary>
```

---

## What Error Boundaries Do NOT Catch
- Errors in **event handlers** (use `try/catch` instead).
- **Async** errors (promises, `setTimeout`).
- **Server-side** rendering errors.
- Errors in the **error boundary component itself**.

---

## Related Topics
- [[Error handling MOC]]
- [[Fallback UI patterns]]
- [[Observability and monitoring MOC]]
