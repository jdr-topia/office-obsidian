---
created: 2026-04-01
tags: [concept]
related: [ [[Requirement analysis in frontend design]], [[Functional vs non-functional requirements]], [[Core web vitals]], [[Frontend system design MOC]] ]
status: active
---

# Performance requirements

## Overview

Performance requirements define measurable targets for how fast and responsive your frontend must be. Unlike vague goals like "the app should be fast," good performance requirements are **specific, measurable, and tied to real user impact**.

Performance is not just a nice-to-have. It is directly correlated with user retention, conversion rates, and SEO rankings.

> **Google's research**: A 100ms delay in mobile page load time can decrease conversion rates by up to 8%.

---

## The Core Performance Model: User-Centric Metrics

Rather than measuring raw server response times, modern performance is measured by how the user **perceives** the experience.

### The Core Web Vitals (CWV)

Google's official metrics for real-world page experience:

| Metric | Full Name | Measures | Good Target |
| :--- | :--- | :--- | :--- |
| **LCP** | Largest Contentful Paint | Loading performance | < 2.5s |
| **CLS** | Cumulative Layout Shift | Visual stability | < 0.1 |
| **INP** | Interaction to Next Paint | Responsiveness | < 200ms |

> **Note**: INP replaced FID (First Input Delay) in March 2024 as the official Core Web Vitals responsiveness metric.

→ Deep dive: [[Core web vitals]]

---

## Other Important Metrics

| Metric | Measures | Why it Matters |
| :--- | :--- | :--- |
| **TTFB** (Time to First Byte) | Server response speed | Slow TTFB kills SSR performance |
| **FCP** (First Contentful Paint) | First pixel rendered on screen | Impacts perceived load speed |
| **TTI** (Time to Interactive) | When the page is fully interactive | Key for JS-heavy SPAs |
| **TBT** (Total Blocking Time) | Time JS blocks the main thread | Predicts INP |

---

## How to Define Performance Requirements

When gathering requirements, ask these questions:

1. **Who is the target user?** Mobile on 3G, or desktop on fibre?
2. **What is the geography?** Users in India face higher latency than in the US.
3. **What is the business context?** E-commerce needs faster LCP than internal tools.
4. **What is the acceptable budget?** JS bundle size limit, image size limit.

### Performance Budget Example

A performance budget is a set of explicit constraints for your project:

```
Performance Budget: E-commerce Product Page
-----------------------------------------------
Total JS bundle size:     < 200 KB (gzipped)
Hero image size:          < 100 KB (WebP)
LCP target:               < 2.5 seconds (4G mobile)
CLS target:               < 0.05
INP target:               < 150ms
TTFB target:              < 600ms
Lighthouse score:         > 90 (Performance)
```

---

## Defining Performance Requirements in Practice

### For a public-facing marketing site
```
- LCP < 2.5s on 4G mobile (primary user segment)
- CLS < 0.1 (no layout shifting during font/image load)
- No JS required for above-the-fold content (SSG preferred)
- All images: WebP format, responsive srcset
```

### For an internal admin dashboard
```
- TTFB < 200ms (internal network, server-side data fetch)
- TTI < 3s on first load (complex React app is acceptable)
- Table must render 5,000 rows without lag (virtualization required)
- Data refreshes every 30 seconds without full page reload
```

---

## Tools for Measuring Performance

### During development
- **Chrome DevTools → Performance tab**: Record runtime traces, find long tasks.
- **Lighthouse**: Audits LCP, CLS, INP, TBT, and gives a score.
- **`next/bundle-analyzer`**: Visualize your Next.js bundle size.

```bash
# Install bundle analyser for Next.js
npm install --save-dev @next/bundle-analyzer
```

```javascript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
})

module.exports = withBundleAnalyzer({})
```

```bash
ANALYZE=true npm run build
```

### In production (Real User Monitoring)
- **Vercel Analytics**: Real User Monitoring (RUM) built into Vercel deployments.
- **web-vitals library**: Report CWV from real user sessions.

```javascript
// app/layout.tsx
import { useReportWebVitals } from 'next/web-vitals'

export function MyApp() {
  useReportWebVitals((metric) => {
    console.log(metric) // LCP, CLS, INP, TTFB, FCP
    // Send to your analytics service
  })
}
```

- **Google Search Console**: Free field data on CWV from real users.

---

## Common Performance Traps in Next.js

| Trap | Problem | Fix |
| :--- | :--- | :--- |
| No `next/image` | Unoptimized image sizes | Use `<Image>` component |
| All components are Client Components | Large JS bundle sent to browser | Default to Server Components |
| No code splitting | Full bundle loaded upfront | Use dynamic imports |
| No `loading` state | Layout shift on data load | Use Suspense boundaries |

---

## Related Notes
- [[Core web vitals]]
- [[Code splitting and lazy loading]]
- [[Image and font optimization]]
- [[HTTP caching and prefetching]]
- [[Functional vs non-functional requirements]]
