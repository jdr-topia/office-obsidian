---
created: 2026-04-07
tags: [moc, observability, monitoring, logging]
related: [ [[Frontend system design MOC]], [[Error handling MOC]] ]
status: active
---

# Observability and monitoring MOC

Observability is your ability to understand what your application is doing in production. Without it, you are flying blind — bugs, regressions, and performance issues go undetected until users complain.

## The Three Pillars of Observability

| Pillar | What it captures | Frontend Tools |
|---|---|---|
| **Logs** | Discrete events and errors | Sentry, Datadog, LogRocket |
| **Metrics** | Aggregated measurements over time | Web Vitals, Performance API |
| **Traces** | Request lifecycle across systems | OpenTelemetry, Datadog APM |

## Notes in this Section
- [[Frontend error monitoring]]: Capturing, grouping, and alerting on runtime errors.
- [[Performance monitoring]]: Tracking Core Web Vitals and UX metrics in production.
- [[Feature flags and A/B testing]]: Safely rolling out features and running experiments.

## Why Observability Matters

You cannot optimize or fix what you cannot measure. Observability closes the gap between development and production by answering:
- Is this new feature causing errors?
- Did this deploy regress our LCP?
- What percentage of users hit this edge case?

---
## Related Topics
- [[Frontend system design MOC]]
- [[Error handling MOC]]
- [[Performance optimization MOC]]
- [[Testing strategy MOC]]
