---
created: 2026-04-07
tags: [testing, rtl, react, component-testing]
related: [ [[Testing strategy MOC]], [[Unit testing with Jest and Vitest]], [[E2E testing with Playwright]] ]
status: active
---

# Component testing with React Testing Library

React Testing Library (RTL) is the standard for testing React components. Its guiding philosophy is to test components from the **user's perspective** — interacting with the DOM the way a user would, not by inspecting implementation details.

## The Core Philosophy

> **"The more your tests resemble the way your software is used, the more confidence they give you."** — Kent C. Dodds

This means:
- ❌ Do NOT test internal state or implementation details.
- ✅ DO test what the user sees and what happens when they interact with it.

---

## Querying Elements (Priority Order)

RTL provides queries that mirror how users and assistive technologies find elements. Use them in this priority order:

| Priority | Query | When to Use |
|---|---|---|
| 1 | `getByRole` | Interactive elements (button, link, checkbox) |
| 2 | `getByLabelText` | Form inputs associated with a label |
| 3 | `getByPlaceholderText` | Inputs with a placeholder |
| 4 | `getByText` | Non-interactive text content |
| 5 | `getByDisplayValue` | Current value of form elements |
| 6 | `getByTestId` | Last resort — use `data-testid` attribute |

---

## Writing Component Tests

```tsx
// components/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { LoginForm } from './LoginForm';

describe('LoginForm', () => {
  it('renders the email and password fields', () => {
    render(<LoginForm onSubmit={vi.fn()} />);

    expect(screen.getByLabelText(/email/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/password/i)).toBeInTheDocument();
  });

  it('calls onSubmit with correct values when the form is submitted', async () => {
    const user = userEvent.setup();
    const handleSubmit = vi.fn();
    render(<LoginForm onSubmit={handleSubmit} />);

    await user.type(screen.getByLabelText(/email/i), 'alice@example.com');
    await user.type(screen.getByLabelText(/password/i), 'password123');
    await user.click(screen.getByRole('button', { name: /log in/i }));

    expect(handleSubmit).toHaveBeenCalledWith({
      email: 'alice@example.com',
      password: 'password123',
    });
  });

  it('shows a validation error for an empty email', async () => {
    const user = userEvent.setup();
    render(<LoginForm onSubmit={vi.fn()} />);

    await user.click(screen.getByRole('button', { name: /log in/i }));

    expect(screen.getByText(/email is required/i)).toBeInTheDocument();
  });
});
```

---

## Testing Async Behavior

Use `waitFor` or `findBy*` queries for async UI updates:

```tsx
it('shows a success message after saving', async () => {
  // Mock the API call
  vi.mocked(saveProfile).mockResolvedValue({ success: true });

  render(<ProfileEditor />);
  await userEvent.click(screen.getByRole('button', { name: /save/i }));

  // findBy* waits for the element to appear (async)
  expect(await screen.findByText(/saved successfully/i)).toBeInTheDocument();
});
```

---

## Wrapping with Providers

Real components often need context providers. Create a custom `render` utility:

```tsx
// test-utils.tsx
import { render } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { SessionProvider } from 'next-auth/react';

function AllProviders({ children }) {
  const queryClient = new QueryClient({ defaultOptions: { queries: { retry: false } } });
  return (
    <SessionProvider session={null}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </SessionProvider>
  );
}

// Custom render that wraps with all providers
const customRender = (ui, options) => render(ui, { wrapper: AllProviders, ...options });
export { customRender as render };
export * from '@testing-library/react';
```

---

## Related Topics
- [[Testing strategy MOC]]
- [[Unit testing with Jest and Vitest]]
- [[E2E testing with Playwright]]
