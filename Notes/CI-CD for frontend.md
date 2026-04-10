---
created: 2026-04-10
tags: [build, ci-cd, deployment, github-actions]
related: [ [[Build and deployment MOC]], [[E2E testing with Playwright]], [[Edge deployment and CDN]] ]
status: active
---

# CI/CD for frontend

**Continuous Integration (CI)** automatically validates every code change. **Continuous Deployment (CD)** automatically ships passing builds to users. Together they form a safety net that prevents regressions from reaching production.

## The Frontend CI Pipeline

```
git push / PR opened
  └─ 1. Install dependencies (npm ci)
  └─ 2. Static analysis (TypeScript, ESLint)
  └─ 3. Unit & component tests (Vitest)
  └─ 4. Build (next build)
  └─ 5. E2E tests (Playwright) — against preview deployment
  └─ 6. Deploy to production (on merge to main)
```

---

## GitHub Actions Example (Next.js)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Type check
        run: npx tsc --noEmit

      - name: Lint
        run: npm run lint

      - name: Unit tests
        run: npm run test

      - name: Build
        run: npm run build

  e2e:
    needs: validate
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: 'npm' }
      - run: npm ci
      - name: Install Playwright browsers
        run: npx playwright install --with-deps chromium
      - name: Run E2E tests
        run: npx playwright test
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: playwright-report/
```

---

## Preview Deployments

Preview deployments create a unique, shareable URL for every Pull Request. This is one of the most powerful patterns in modern frontend development.

```
PR #42 → Vercel builds → https://my-app-pr-42.vercel.app
```

**Benefits:**
- Designers and PMs can review UI changes without a local setup.
- E2E tests can run against a real deployed environment.
- Staging and production remain clean.

Tools: **Vercel**, **Netlify**, **Cloudflare Pages** — all support this natively.

---

## Environment Variables Management

| Environment | Strategy |
|---|---|
| **Local** | `.env.local` (gitignored) |
| **Preview** | Set in Vercel/Netlify dashboard per branch |
| **Production** | Set in CI/CD secrets or platform dashboard |

```bash
# .env.local (never commit this)
NEXT_PUBLIC_API_URL=http://localhost:4000
DATABASE_URL=postgresql://localhost/mydb

# .env.example (commit this — documents needed vars)
NEXT_PUBLIC_API_URL=
DATABASE_URL=
```

---

## Deployment Checklist

- [ ] `npm ci` used (reproducible installs) not `npm install`
- [ ] TypeScript check passes with `--noEmit`
- [ ] All tests pass before merge
- [ ] Build artifacts are cached between CI runs
- [ ] Source maps uploaded to error monitoring (Sentry)
- [ ] Preview deployments trigger on every PR
- [ ] Production deploys only on merge to `main`

---

## Related Topics
- [[Build and deployment MOC]]
- [[E2E testing with Playwright]]
- [[Edge deployment and CDN]]
- [[Developer experience (DX) MOC]]
