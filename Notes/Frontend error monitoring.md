---
created: 2026-04-07
tags: [observability, error-monitoring, sentry, logging]
related: [ [[Observability and monitoring MOC]], [[Error handling MOC]], [[Error boundaries in React]] ]
status: active
---

# Frontend error monitoring

Frontend error monitoring is the practice of automatically capturing, reporting, and alerting on runtime errors that occur in users' browsers in production. It bridges the gap between your test suite and real-world usage.

## What Needs to Be Captured

| Error Type | Source | How to Capture |
|---|---|---|
| JavaScript syntax/runtime errors | Unhandled exceptions | `window.onerror` / `window.addEventListener('error')` |
| Unhandled Promise rejections | Async errors | `window.addEventListener('unhandledrejection')` |
| React render errors | Component crashes | Error Boundaries (`componentDidCatch`) |
| Network/API errors | Failed fetches | Axios interceptors / TanStack Query `onError` |
| Console errors | Dev malpractice | Log drain integrations |

---

## Sentry (Industry Standard)

Sentry is the most widely used frontend error monitoring tool. It captures errors, groups them intelligently, attaches user context, and shows the exact source-mapped stack trace.

### Setup in Next.js

```bash
npm install @sentry/nextjs
npx @sentry/wizard@latest -i nextjs
```

This auto-creates `sentry.client.config.ts`, `sentry.server.config.ts`, and `sentry.edge.config.ts`.

```ts
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  tracesSampleRate: 0.1,     // Capture 10% of transactions for performance tracing
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0, // Always replay sessions where an error occurs
  integrations: [Sentry.replayIntegration()],
});
```

### Capturing Errors Manually

```tsx
import * as Sentry from "@sentry/nextjs";

// In an Error Boundary's componentDidCatch:
componentDidCatch(error, info) {
  Sentry.captureException(error, { extra: info });
}

// In an event handler:
async function handleSubmit() {
  try {
    await submitForm(data);
  } catch (error) {
    Sentry.captureException(error, {
      tags: { feature: 'checkout' },
      user: { id: currentUser.id },
    });
    toast.error("Submission failed. Our team has been notified.");
  }
}
```

---

## Setting User Context

Error reports without user context are hard to triage. Set user info after login so every subsequent error is linked to that user.

```ts
import * as Sentry from "@sentry/nextjs";

// After successful authentication
Sentry.setUser({
  id: user.id,
  email: user.email,
  username: user.name,
});

// On logout
Sentry.setUser(null);
```

---

## Source Maps

Without source maps, stack traces point to minified, unreadable code. Configure your build to upload source maps to Sentry:

```js
// next.config.js (with @sentry/nextjs)
const { withSentryConfig } = require('@sentry/nextjs');

module.exports = withSentryConfig(nextConfig, {
  silent: true,
  widenClientFileUpload: true,
  hideSourceMaps: true, // Don't expose source maps to end users
});
```

---

## Alert Strategy

- **Spike Alert**: Notify when a new error's occurrence rate exceeds a threshold.
- **Regression Alert**: Notify when a previously resolved error recurs after a deploy.
- **First Seen**: Notify immediately when a completely new error is seen.

---

## Related Topics
- [[Observability and monitoring MOC]]
- [[Error handling MOC]]
- [[Error boundaries in React]]
- [[API error handling]]
