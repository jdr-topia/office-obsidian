---
created: 2026-04-07
tags: [moc, accessibility, a11y]
related: [ [[Frontend system design MOC]], [[Semantic HTML]] ]
status: active
---

# Accessibility (a11y) MOC

Accessibility ensures that your application is usable by everyone, including people with visual, motor, auditory, and cognitive disabilities. It is not a feature — it is a quality standard.

## Core Principles (WCAG — POUR)

| Principle | Meaning |
|---|---|
| **Perceivable** | Users can perceive the UI (alt text, captions) |
| **Operable** | Users can operate the UI (keyboard, no timing traps) |
| **Understandable** | UI behavior is predictable and errors are clear |
| **Robust** | Works with current and future assistive technologies |

## Notes in this Section
- [[Semantic HTML]]: Using the right HTML element for the right job.
- [[ARIA attributes]]: Enhancing semantics where HTML falls short.
- [[Keyboard navigation and focus management]]: Ensuring full keyboard operability.

## Tooling
- **axe DevTools** / **Lighthouse**: Automated a11y auditing.
- **NVDA / VoiceOver**: Screen reader testing.
- **`eslint-plugin-jsx-a11y`**: Lint a11y issues in JSX at build time.

---
## Related Topics
- [[Frontend system design MOC]]
- [[Headless components]] (headless libs bake in a11y automatically)
- [[Error handling MOC]] (error messages must be accessible)
