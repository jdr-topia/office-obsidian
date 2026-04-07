---
created: 2026-04-07
tags: [moc, testing, quality]
related: [ [[Frontend system design MOC]], [[Developer experience (DX) MOC]] ]
status: active
---

# Testing strategy MOC

A testing strategy defines what to test, at what level, and with which tools. The goal is maximum confidence for minimum maintenance burden.

## The Testing Trophy (Frontend Perspective)

Kent C. Dodds' model, adapted for frontend (replacing the classic pyramid):

```
           /‾‾‾‾‾‾‾‾\
          /  E2E (few) \       ← Playwright, Cypress
         /‾‾‾‾‾‾‾‾‾‾‾‾‾\
        /  Integration   \     ← React Testing Library
       /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
      /   Unit (targeted)  \   ← Jest, Vitest
     /‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾\
    /    Static Analysis     \  ← TypeScript, ESLint
   ‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾
```

**The widest, most valuable layer is Integration testing** — testing how components behave together from the user's perspective.

## Notes in this Section
- [[Unit testing with Jest and Vitest]]: Testing isolated functions and logic.
- [[Component testing with React Testing Library]]: Testing UI from the user's perspective.
- [[E2E testing with Playwright]]: Testing complete user flows in a real browser.

## Guiding Principle

> **"Write tests. Not too many. Mostly integration."** — Kent C. Dodds

---
## Related Topics
- [[Frontend system design MOC]]
- [[Developer experience (DX) MOC]]
- [[Observability and monitoring MOC]]
