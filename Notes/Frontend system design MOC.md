---
created: 2026-04-01
tags: [moc]
related: [ [[Frontend MOC]] ]
status: active
---

# Frontend system design MOC

A comprehensive map of all topics required to design scalable, performant, and maintainable frontend systems. This is your entry point to the entire knowledge base.

---

## 1. Requirement analysis
> Understand what you're building before you build it.
- [[Requirement analysis in frontend design]]
- [[Functional vs non-functional requirements]]
- [[Performance requirements]]
- [[Scalability in frontend systems]]
- [[Accessibility requirements]]
- [[SEO considerations in frontend design]]
- [[Internationalization (i18n)]]

## 2. High level architecture
> The blueprint of how the system is structured.
- [[High level architecture MOC]]
- [[Monolith vs micro-frontend]]
- [[SPA vs MPA]]
- [[Backend for Frontend (BFF)]]
- [[CDN usage in frontend]]
- [[API gateway integration]]

## 3. Rendering strategies
> How the UI is generated and delivered.
- [[Rendering strategies MOC]]
- [[Client side rendering (CSR)]]
- [[Server side rendering (SSR)]]
- [[Static site generation (SSG)]]
- [[Incremental static regeneration (ISR)]]
- [[Streaming SSR]]
- [[Hydration and partial hydration]]
- [[Islands architecture]]

## 4. State management
> How data flows and is shared across your application.
- [[State management MOC]]
- [[Local state in React]]
- [[Global state management]]
- [[Server state management]]
- [[Derived state and memoization]]

## 5. Data fetching strategy
> How your frontend talks to backend APIs.
- [[Data fetching strategy MOC]]
- [[REST API vs GraphQL in frontend]]
- [[Pagination and infinite scrolling]]
- [[Optimistic updates and retry strategies]]
- [[REST API integration]]
- [[GraphQL integration]]
- [[Request deduplication and batching]]

## 6. Performance optimization
> Making your app fast for real users.
- [[Performance optimization MOC]]
- [[Core web vitals and metrics]]
- [[Code and bundle optimization]]
- [[Asset optimization in Next.js]]
- [[Network optimization patterns]]
- [[Virtualization]]
- [[Image and font optimization]]
- [[HTTP caching and prefetching]]

## 7. Scalability
> Building for large teams and large traffic.
- [[Scalability MOC]]
- [[Monorepo architecture]]
- [[Module federation]]
- [[Design systems and shared libraries]]

## 8. UI architecture
> How UI components are designed and composed.
- [[UI architecture MOC]]
- [[Atomic design pattern]]
- [[Container vs presentational components]]
- [[Headless components]]
- [[Compound components]]

## 9. Routing strategy
> How navigation works in your application.
- [[Routing strategy MOC]]
- [[Client side routing]]
- [[Nested and dynamic routes]]
- [[Protected routes]]
- [[Route based code splitting]]

## 10. Authentication and authorization
> Securing your frontend application.
- [[Authentication and authorization MOC]]
- [[Session vs JWT authentication]]
- [[OAuth and social login]]
- [[RBAC in frontend]]
- [[Authentication with NextAuth]]

## 11. Error handling
> Building resilient frontend systems.
- [[Error handling MOC]]
- [[Error boundaries in React]]
- [[API error handling]]
- [[Fallback UI patterns]]

## 12. Caching strategies
> Reducing redundant work and API calls.
- [[Caching strategies MOC]]
- [[Browser and HTTP cache]]
- [[Service worker cache]]
- [[Stale-while-revalidate]]

## 13. Real time communication
> Delivering live data to the user.
- [[Real time communication MOC]]
- [[WebSockets]]
- [[Server sent events (SSE)]]
- [[Polling and long polling]]

## 14. Accessibility (a11y)
> Making your UI usable by everyone.
- [[Accessibility (a11y) MOC]]
- [[Semantic HTML]]
- [[ARIA attributes]]
- [[Keyboard navigation and focus management]]

## 15. Security
> Protecting your users and your application.
- [[Frontend security MOC]]
- [[XSS protection]]
- [[CSRF protection]]
- [[Content Security Policy (CSP)]]

## 16. Internationalization (i18n)
> Supporting a global audience.
- [[Internationalization (i18n)]]
- [[RTL support]]
- [[Translation loading strategies]]

## 17. Offline support
> Handling network failures gracefully.
- [[Offline support MOC]]
- [[Service workers]]
- [[Progressive Web Apps (PWA)]]

## 18. Observability and monitoring
> Understanding what your app does in production.
- [[Observability and monitoring MOC]]
- [[Frontend error monitoring]]
- [[Performance monitoring]]
- [[Feature flags and A/B testing]]

## 19. Testing strategy
> Ensuring correctness and confidence.
- [[Testing strategy MOC]]
- [[Unit testing with Jest and Vitest]]
- [[Component testing with React Testing Library]]
- [[E2E testing with Playwright]]

## 20. Build and deployment
> Shipping your app to users.
- [[Build and deployment MOC]]
- [[Vite vs Webpack vs Turbopack]]
- [[CI/CD for frontend]]
- [[Edge deployment and CDN]]

## 21. Developer experience (DX)
> Making the team productive.
- [[Developer experience (DX) MOC]]
- [[TypeScript in frontend]]
- [[Linting and code formatting]]
- [[Git hooks and commit linting]]

## 22. Design systems
> Reusable UI at scale.
- [[Design systems MOC]]
- [[Component libraries]]
- [[Design tokens and theming]]
- [[Storybook]]

---
## Related Maps
- [[Frontend MOC]]
