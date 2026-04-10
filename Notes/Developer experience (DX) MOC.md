---
created: 2026-04-10
tags: [moc, dx, developer-experience, tooling]
related: [ [[Frontend system design MOC]], [[Build and deployment MOC]], [[Testing strategy MOC]] ]
status: active
---

# Developer experience (DX) MOC

Developer Experience (DX) encompasses every tool, workflow, and practice that impacts how fast and confidently a developer can build, test, and ship features. Good DX reduces friction, catches bugs early, and lets teams scale without chaos.

## The DX Stack

```
Write code    → TypeScript (type safety at authoring time)
Commit code   → ESLint + Prettier (consistency)
              → husky + lint-staged (automated enforcement)
Review code   → PR previews, conventional commits
Test code     → Vitest + Playwright (confidence)
Ship code     → CI/CD, feature flags (safe delivery)
```

## Notes in this Section
- [[TypeScript in frontend]]: Why TypeScript is essential for scalable codebases.
- [[Linting and code formatting]]: ESLint, Prettier, and automated enforcement.
- [[Git hooks and commit linting]]: Husky, lint-staged, and conventional commits.

## Why DX Matters at Scale

- **Onboarding**: Good DX means a new developer can be productive on day one.
- **Velocity**: Automated checks catch issues instantly rather than in code review.
- **Quality**: Consistent formatting and type safety prevent entire classes of bugs.

---
## Related Topics
- [[Frontend system design MOC]]
- [[Build and deployment MOC]]
- [[Testing strategy MOC]]
- [[CI/CD for frontend]]
