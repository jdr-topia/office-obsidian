---
created: 2026-04-02
tags: [moc]
related: [ [[Frontend system design MOC]], [[Performance requirements]], [[Core web vitals and metrics]] ]
status: active
---

# Performance optimization MOC

## Overview

Performance optimization is a collection of strategies, tools, and technical decisions aimed at making your frontend application **fast, responsive, and visually stable**.

It's not just a "nice-to-have" feature; it's a **core requirement** for user retention, conversion rates, and SEO.

---

## The Four Pillars of Optimization

### 1. Code & Bundle Optimization
Reducing the amount of JavaScript sent to the browser and ensuring it executes efficiently.
- **Code Splitting**: Loading only necessary code per page.
- **Tree Shaking**: Removing unused code from final bundles.
- **Budgeting**: Enforcing size limits for your JS.
→ [[Code and bundle optimization]]

### 2. Network & Delivery Optimization
Improving how assets are delivered across the internet.
- **CDN Usage**: Global distribution.
- **HTTP Caching**: Reusing resources (`ETags`, `Cache-Control`).
- **Prefetching/Preloading**: Loading future-needed assets in advance.
→ [[Network optimization patterns]]

### 3. Rendering & Layout Optimization
Reducing browser work and preventing visual instability.
- **Core Web Vitals**: Measuring the real user experience.
- **Virtualization**: Rendering large lists incrementally.
- **Memoization**: Preventing unnecessary React re-renders.
→ [[Core web vitals and metrics]]
→ [[Derived state and memoization]]

### 4. Asset Optimization
Delivering high-quality images and fonts without excessive bandwidth.
- **Image Optimization**: WebP, lazy loading, responsive sizes.
- **Font Optimization**: WOOW2, `font-display: swap`.
→ [[Asset optimization in Next.js]]

---

## Essential Metrics

| Metric | Short Name | Targeted Success |
| :--- | :--- | :--- |
| **Largest Contentful Paint** | LCP | < 2.5s |
| **Interaction to Next Paint** | INP | < 200ms |
| **Cumulative Layout Shift** | CLS | < 0.1 |

→ [[Core web vitals and metrics]]

---

## Measuring Tools

### Laboratory Tools (Local Audits)
- **Lighthouse**: Integrated into Chrome DevTools.
- **Chrome DevTools Performance Tab**: Visualizes the main-thread timeline.
- **`next/bundle-analyzer`**: Visualizes JS bundle composition.

### Field Tools (Real User Monitoring - RUM)
- **Google Search Console**: Aggregated CWV data from real users.
- **Vercel Analytics**: Built-in RUM for Next.js apps.
- **PageSpeed Insights**: Detailed field + laboratory reports.

---

## Related Notes
- [[Performance requirements]]
- [[Core web vitals and metrics]]
- [[Code and bundle optimization]]
- [[Asset optimization in Next.js]]
- [[Network optimization patterns]]
- [[Frontend system design MOC]]
