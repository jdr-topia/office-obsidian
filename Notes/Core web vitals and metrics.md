---
created: 2026-04-02
tags: [concept, metric]
related: [ [[Performance optimization MOC]], [[Performance requirements]], [[Frontend system design MOC]] ]
status: active
---

# Core web vitals and metrics

## Overview

**Core Web Vitals (CWV)** are a set of specific factors that Google considers important for a webpage’s overall user experience. They are **user-centric metrics** that help quantify the real-world experience of your site.

Google uses these metrics as **actual ranking factors** in its search algorithm.

---

## 1. Largest Contentful Paint (LCP)

LCP measures **loading performance**. It marks the time when the largest image or text block in the viewport (above the fold) is rendered.

- **Objective**: Get content to the user as fast as possible.
- **Good Target**: < 2.5 seconds.
- **Common LCP Elements**: Hero image, large heading (`h1`), main body text.

### How to Optimize LCP
- **Use `next/image` with `priority`**: Next.js preloads priority images.
- **Reduce TTFB**: Use Server Side Rendering (SSR) effectively or Static Site Generation (SSG) with a CDN.
- **Eliminate Render-Blocking JS/CSS**: Inline critical CSS, defer non-critical JS.
- **CDN Usage**: Move assets closer to the user.

---

## 2. Interaction to Next Paint (INP)

INP measures **responsiveness**. It replaced First Input Delay (FID) in March 2024. It measures the latency of **every** interaction (clicks, taps, and keyboard presses) throughout the user's visit.

- **Objective**: Ensure the UI responds instantly to user input.
- **Good Target**: < 200 milliseconds.

### How to Optimize INP
- **Avoid Main-Thread Blocking**: Break up "long tasks" using `setTimeout` or `requestIdleCallback`.
- **Optimize Event Handlers**: Don't perform heavy calculations inside an `onClick`.
- **Use Web Workers**: Offload complex tasks to a separate thread.
- **React Transitions**: Use `useTransition` to let React prioritize high-priority input (like typing) over lower-priority updates (like filtering a long list).

---

## 3. Cumulative Layout Shift (CLS)

CLS measures **visual stability**. It quantifies how much unexpected movement of page content occurs during the page lifecycle.

- **Objective**: Prevent elements from jumping around while the user is reading or clicking.
- **Good Target**: < 0.1.

### How to Optimize CLS
- **Reserve Space for Images/Videos**: Always specify `width` and `height` attributes (Next.js `<Image>` does this automatically).
- **Use Aspect Ratio**: `aspect-ratio: 16 / 9` in CSS.
- **Avoid Injecting Content Above Current Content**: Don't insert banners or ads at the top of the page after the initial paint.
- **Font Optimization**: Use `font-display: swap` to prevent "FOUT" (Flash of Unstyled Text).

---

## 4. Other Key Metrics

| Metric | Short Name | Significance |
| :--- | :--- | :--- |
| **Time to First Byte** | TTFB | Measures server response speed (DNS + Handshake + Processing). Goal: < 800ms. |
| **First Contentful Paint** | FCP | Time when the first pixel is painted on the screen. Goal: < 1.8s. |
| **Time to Interactive** | TTI | When the page is fully interactive (main thread is quiet). |
| **Total Blocking Time** | TBT | Sum of all time the main thread was blocked between FCP and TTI. |

---

## Measuring and Monitoring CWV

### Laboratory Data (Lighthouse)
Simulates a user visit in a controlled environment (often on a "Mobile 3G" or "Desktop" profile).
- **Tool**: Chrome DevTools > Lighthouse.
- **Best For**: Debugging specific performance issues before deployment.

### Field Data (RUM - Real User Monitoring)
Captures the actual experience of real users across different devices, networks, and locations.
- **Tool**: Google Search Console, Vercel Analytics, or the `web-vitals` library.
- **Best For**: Understanding the actual user experience and finding "hidden" bottlenecks.

---

## Related Notes
- [[Performance optimization MOC]]
- [[Performance requirements]]
- [[Next.js Rendering Strategies (Complete Guide)]]
- [[Frontend system design MOC]]
