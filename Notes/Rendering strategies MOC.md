---
created: 2026-04-01
tags: [moc]
related: [ [[Frontend system design MOC]], [[SPA vs MPA]], [[SEO considerations in frontend design]], [[Next.js Rendering Strategies (Complete Guide)]] ]
status: active
---

# Rendering strategies MOC

## Overview

Rendering strategy is the decision of **where and when** your HTML is generated. It is one of the most impactful architectural decisions because it determines your SEO capability, initial load performance, and server costs.

> In modern Next.js (App Router), rendering is **not page-level** — it is **component-level**. You can mix SSR, SSG, CSR, and Streaming within a single page.

---

## The Spectrum

```
Build Time ←——————————————————————————→ Runtime (Per Request)
   │                                          │
  SSG          ISR          SSR           CSR
(Fastest)                             (Most Dynamic)
```

---

## All Rendering Strategies

### Static (Build Time)
- [[Static site generation (SSG)]] — HTML generated once at build. Served from CDN. Fastest possible.
- [[Incremental static regeneration (ISR)]] — SSG + automatic background regeneration after a time interval.

### Dynamic (Per Request)
- [[Server side rendering (SSR)]] — HTML generated on every request on the server. Always fresh.
- [[Client side rendering (CSR)]] — HTML generated in the browser by JavaScript. No server involvement.

### Advanced (Hybrid & Progressive)
- [[Streaming SSR]] — Send HTML in chunks as it becomes ready. Faster perceived load.
- [[Hydration and partial hydration]] — How the server-rendered HTML becomes interactive in the browser.
- [[Islands architecture]] — Mostly static HTML with isolated interactive "islands" of JavaScript.

---

## Full Reference: Next.js Rendering Strategies (Complete Guide)
→ [[Next.js Rendering Strategies (Complete Guide)]] — detailed code examples for all strategies

---

## Decision Tree

```
Is the page public-facing (SEO needed)?
  ├─ Yes → Does data change frequently?
  │           ├─ No → SSG
  │           ├─ Periodically → ISR
  │           └─ Yes, per request → SSR / Streaming SSR
  └─ No (behind login) → CSR is acceptable
```

## Quick Comparison

| Strategy | Speed | SEO | Data Freshness | Server Cost |
| :--- | :--- | :--- | :--- | :--- |
| **SSG** | Fastest | ✅ Excellent | ❌ Stale | Lowest |
| **ISR** | Fast | ✅ Excellent | ⚠️ Configurable | Low |
| **SSR** | Medium | ✅ Excellent | ✅ Always fresh | High |
| **CSR** | Slow initial | ❌ Poor | ✅ Always fresh | Lowest |
| **Streaming SSR** | Fast perceived | ✅ Excellent | ✅ Always fresh | Medium |

---

## Related Notes
- [[SPA vs MPA]]
- [[SEO considerations in frontend design]]
- [[React server components]]
- [[Performance requirements]]
- [[Frontend system design MOC]]
