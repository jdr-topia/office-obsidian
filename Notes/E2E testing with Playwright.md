---
created: 2026-04-07
tags: [testing, e2e, playwright, automation]
related: [ [[Testing strategy MOC]], [[Component testing with React Testing Library]], [[CI/CD for frontend]] ]
status: active
---

# E2E testing with Playwright

End-to-End (E2E) tests verify complete user flows in a real browser — from loading the page to completing a multi-step interaction. Playwright is the industry-leading E2E testing tool, offering cross-browser support, auto-waits, and a powerful API.

## Why Playwright Over Cypress?

| Feature | Playwright | Cypress |
|---|---|---|
| **Browser support** | Chromium, Firefox, WebKit (Safari) | Chromium-based only |
| **Parallelism** | Native, fast | Slower without cloud plan |
| **Multiple tabs/origins** | ✅ Supported | ❌ Limited |
| **Network interception** | ✅ Full control | ✅ Good |
| **Auto-wait** | ✅ All actions auto-wait | ✅ Good |
| **TypeScript** | First-class | Good |

---

## Setup

```bash
npm install -D @playwright/test
npx playwright install  # Installs browser binaries
```

```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  retries: process.env.CI ? 2 : 0,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry', // Record trace on failure
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    { name: 'Mobile Safari', use: { ...devices['iPhone 13'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## Writing Tests

```ts
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('user can log in with valid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.getByLabel('Email').fill('alice@example.com');
    await page.getByLabel('Password').fill('password123');
    await page.getByRole('button', { name: 'Log in' }).click();

    // Playwright auto-waits for navigation
    await expect(page).toHaveURL('/dashboard');
    await expect(page.getByText('Welcome back, Alice')).toBeVisible();
  });

  test('shows an error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.getByLabel('Email').fill('wrong@example.com');
    await page.getByLabel('Password').fill('wrongpassword');
    await page.getByRole('button', { name: 'Log in' }).click();

    await expect(page.getByText('Invalid email or password')).toBeVisible();
    await expect(page).toHaveURL('/login'); // No redirect
  });
});
```

---

## Network Interception (Mocking APIs)

```ts
test('shows a list of posts from the API', async ({ page }) => {
  // Intercept the API call and return mock data
  await page.route('/api/posts', (route) =>
    route.fulfill({
      status: 200,
      contentType: 'application/json',
      body: JSON.stringify([
        { id: 1, title: 'Hello World' },
        { id: 2, title: 'Second Post' },
      ]),
    })
  );

  await page.goto('/posts');

  await expect(page.getByText('Hello World')).toBeVisible();
  await expect(page.getByText('Second Post')).toBeVisible();
});
```

---

## Key Best Practices

1. **Use `data-testid` sparingly** — prefer `getByRole`, `getByLabel` (same philosophy as RTL).
2. **Test user journeys, not implementation**: Test "checkout flow", not "the button has class 'btn-primary'".
3. **Keep E2E tests few and high-value**: They are slow and brittle. Focus on critical paths only (login, checkout, onboarding).
4. **Use fixtures for auth state**: Share login state across tests with `storageState` to avoid re-logging in every test.

```ts
// Save logged-in state once, reuse across all tests
// playwright.config.ts
use: {
  storageState: 'e2e/.auth/user.json',
},
```

---

## Run Commands

```bash
npx playwright test                    # Run all E2E tests
npx playwright test --ui               # Open interactive UI mode
npx playwright test auth.spec.ts       # Run a single spec
npx playwright show-report             # Open HTML report
```

---

## Related Topics
- [[Testing strategy MOC]]
- [[Component testing with React Testing Library]]
- [[CI/CD for frontend]]
