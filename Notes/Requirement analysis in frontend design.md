---
created: 2026-04-01
tags: [moc, concept]
related: [ [[Frontend system design MOC]], [[Functional vs non-functional requirements]], [[Performance requirements]], [[Scalability in frontend systems]], [[Accessibility requirements]], [[SEO considerations in frontend design]], [[Internationalization (i18n)]] ]
status: active
---

# Requirement analysis in frontend design

## Overview

Requirement analysis is the first and most critical step in frontend system design. Before writing a single line of code or choosing a framework, you need to deeply understand **what** the system must do and **how well** it must do it.

Skipping this phase is one of the biggest mistakes teams make. Poor requirement analysis leads to systems that are over-engineered for their actual needs, or collapse under real-world usage.

---

## Why it matters

When designing a frontend system, requirements determine almost every technical decision:

| Requirement | Technical Decision |
| :--- | :--- |
| 1M concurrent users | Need caching, CDN, lazy loading |
| Real-time updates | Need WebSockets or SSE |
| Must rank on Google | Need SSR or SSG, not pure CSR |
| Multi-language support | Need i18n library from day one |
| Accessibility compliance | Need semantic HTML and ARIA roles |

Getting these wrong early is extremely expensive to fix later.

---

## The Two Categories

All requirements fall into two broad buckets. See the deep-dive note for full detail:

→ [[Functional vs non-functional requirements]]

---

## Checklist: Questions to Ask Before Designing

### Functional Questions
- [ ] What are the core user journeys? (e.g., login → dashboard → action)
- [ ] What data is displayed and how frequently does it change?
- [ ] What actions can users take? (CRUD, forms, uploads, etc.)
- [ ] Are there different user roles with different access levels?
- [ ] What external services does the app communicate with?

### Non-Functional Questions
- [ ] What is the expected number of concurrent users?
- [ ] What are acceptable page load time targets (LCP, TTFB)?
- [ ] Does the app need to work offline?
- [ ] What browsers/devices must be supported?
- [ ] Are there legal compliance requirements (GDPR, WCAG)?
- [ ] What are the SEO requirements?
- [ ] Does the app need to support multiple languages?

---

## Example: Analysing a Dashboard

**Requirement**: Design a dashboard that supports 1M users, real-time updates, on both mobile and desktop.

**Analysis Output**:

| Category | Requirement | Technical Implication |
| :--- | :--- | :--- |
| Scale | 1M users | CDN, code splitting, efficient re-rendering |
| Real-time | Live data updates | WebSockets or SSE, optimistic UI |
| Responsive | Mobile + Desktop | CSS Grid/Flexbox, responsive breakpoints |
| Performance | Fast initial load | SSR or SSG for initial HTML |
| Accessibility | General public product | WCAG 2.1 AA compliance |

---

## Sub-Topics (Deep Dives)

- [[Functional vs non-functional requirements]] — What the app does vs. how well it does it
- [[Performance requirements]] — Load time, FPS, Core Web Vitals targets
- [[Scalability in frontend systems]] — Designing for traffic growth
- [[Accessibility requirements]] — WCAG, ARIA, keyboard navigation needs
- [[SEO considerations in frontend design]] — Crawlability, Rendering, metadata
- [[Internationalization (i18n)]] — Multi-language, RTL, locale support
