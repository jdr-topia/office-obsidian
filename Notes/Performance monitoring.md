---
created: 2026-04-07
tags: [observability, performance, web-vitals, metrics]
related: [ [[Observability and monitoring MOC]], [[Core web vitals and metrics]], [[Performance optimization MOC]] ]
status: active
---

# Performance monitoring

Performance monitoring is the practice of continuously measuring your application's real-world speed and user experience metrics in production. Lab tests (Lighthouse) measure potential; production monitoring measures reality.

## Lab vs Field Data

| Type | Where | Tools | Limitation |
|---|---|---|---|
| **Lab** (Synthetic) | Controlled environment | Lighthouse, WebPageTest | Doesn't reflect real users |
| **Field** (Real User Monitoring) | Real browsers, real networks | Web Vitals API, Sentry, Datadog | Requires traffic |

**Always prioritize field data for production decisions.**

---

## The Web Vitals API

The browser natively reports Core Web Vitals. Capture them and send to your analytics.

```ts
// lib/vitals.ts
import { onCLS, onFID, onLCP, onFCP, onTTFB, onINP } from 'web-vitals';

function sendToAnalytics({ name, value, id, rating }) {
  fetch('/api/vitals', {
    method: 'POST',
    body: JSON.stringify({ name, value, id, rating }),
    keepalive: true, // Ensures the request completes even if page unloads
  });
}

export function reportWebVitals() {
  onCLS(sendToAnalytics);
  onFID(sendToAnalytics);
  onLCP(sendToAnalytics);
  onFCP(sendToAnalytics);
  onTTFB(sendToAnalytics);
  onINP(sendToAnalytics);
}
```

### In Next.js (Built-in)

```tsx
// app/layout.tsx or pages/_app.tsx
export function reportWebVitals(metric) {
  // metric: { name, value, id, label, startTime }
  console.log(metric); // Replace with analytics send
}
```

---

## Key Metrics to Track

| Metric | Target | What It Measures |
|---|---|---|
| **LCP** (Largest Contentful Paint) | < 2.5s | Perceived load speed |
| **CLS** (Cumulative Layout Shift) | < 0.1 | Visual stability |
| **INP** (Interaction to Next Paint) | < 200ms | Responsiveness to input |
| **FCP** (First Contentful Paint) | < 1.8s | Initial render speed |
| **TTFB** (Time to First Byte) | < 800ms | Server response speed |

---

## The Performance API

For custom, granular measurements beyond Web Vitals:

```ts
// Mark the start of a critical operation
performance.mark('checkout:start');

await processPayment(orderData);

// Mark the end
performance.mark('checkout:end');
performance.measure('checkout-duration', 'checkout:start', 'checkout:end');

const [measure] = performance.getEntriesByName('checkout-duration');
console.log(`Checkout took: ${measure.duration.toFixed(2)}ms`);

// Send to monitoring
sendToAnalytics({ name: 'checkout-duration', value: measure.duration });
```

---

## Monitoring Tools

| Tool | Strength |
|---|---|
| **Vercel Speed Insights** | Zero-config for Next.js, real user vitals |
| **Sentry Performance** | Tied to error context, trace waterfall |
| **Datadog RUM** | Enterprise, deep session replay & traces |
| **Google Analytics 4** | Free, basic Web Vitals reporting |
| **SpeedCurve** | Competitive benchmarking, budget alerts |

---

## Performance Budgets

Set automated alerts when a metric exceeds a budget to catch regressions before users notice.

```js
// next.config.js
module.exports = {
  experimental: {
    // Fail the build if JS bundle exceeds 200kb
    bundlePagesRouterDependencies: true,
  },
};
```

---

## Related Topics
- [[Observability and monitoring MOC]]
- [[Core web vitals and metrics]]
- [[Performance optimization MOC]]
- [[Network optimization patterns]]
