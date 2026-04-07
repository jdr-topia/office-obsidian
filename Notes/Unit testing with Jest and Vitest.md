---
created: 2026-04-07
tags: [testing, unit-testing, jest, vitest]
related: [ [[Testing strategy MOC]], [[Component testing with React Testing Library]] ]
status: active
---

# Unit testing with Jest and Vitest

Unit testing verifies that an isolated piece of logic (a function, hook, or utility) behaves correctly given specific inputs, independent of any external dependencies.

## Jest vs Vitest

| Feature | Jest | Vitest |
|---|---|---|
| **Ecosystem** | Mature, widely used | Modern, fast-growing |
| **Speed** | Slower (CommonJS transform) | Up to 10× faster (native ESM, Vite-powered) |
| **Config** | Requires Babel/ts-jest for TS | Zero-config for Vite projects |
| **API** | `describe`, `it`, `expect` | Identical API — easy migration |
| **Best For** | CRA, legacy projects | Next.js, Vite, modern stacks |

> **Recommendation**: Use **Vitest** for new Next.js/Vite projects. The API is identical to Jest so knowledge transfers.

---

## Setup (Vitest in Next.js)

```bash
npm install -D vitest @vitejs/plugin-react jsdom @testing-library/react
```

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,           // No need to import describe/it/expect
    setupFiles: './vitest.setup.ts',
  },
});
```

```ts
// vitest.setup.ts
import '@testing-library/jest-dom'; // Adds custom matchers like toBeInTheDocument()
```

---

## What to Unit Test

Unit tests shine for **pure logic** with no UI or side effects:
- Data transformation functions
- Validation logic
- Custom hooks (business logic parts)
- Utility helpers

---

## Writing Tests

### Testing a Pure Function

```ts
// lib/price.ts
export function applyDiscount(price: number, discountPercent: number): number {
  if (discountPercent < 0 || discountPercent > 100) throw new Error('Invalid discount');
  return price * (1 - discountPercent / 100);
}

// lib/price.test.ts
import { describe, it, expect } from 'vitest';
import { applyDiscount } from './price';

describe('applyDiscount', () => {
  it('applies a 20% discount correctly', () => {
    expect(applyDiscount(100, 20)).toBe(80);
  });

  it('returns the original price for 0% discount', () => {
    expect(applyDiscount(100, 0)).toBe(100);
  });

  it('throws for an invalid discount percentage', () => {
    expect(() => applyDiscount(100, -5)).toThrow('Invalid discount');
    expect(() => applyDiscount(100, 110)).toThrow('Invalid discount');
  });
});
```

### Testing a Custom Hook

Use `renderHook` from React Testing Library:

```ts
// hooks/useCounter.test.ts
import { renderHook, act } from '@testing-library/react';
import { useCounter } from './useCounter';

describe('useCounter', () => {
  it('initializes with the given value', () => {
    const { result } = renderHook(() => useCounter(5));
    expect(result.current.count).toBe(5);
  });

  it('increments the count', () => {
    const { result } = renderHook(() => useCounter(0));
    act(() => result.current.increment());
    expect(result.current.count).toBe(1);
  });
});
```

---

## Mocking

```ts
import { vi } from 'vitest';

// Mock a module
vi.mock('@/lib/api', () => ({
  fetchUser: vi.fn().mockResolvedValue({ id: '1', name: 'Alice' }),
}));

// Spy on a function
const consoleSpy = vi.spyOn(console, 'error').mockImplementation(() => {});

// Reset all mocks between tests
afterEach(() => vi.clearAllMocks());
```

---

## Related Topics
- [[Testing strategy MOC]]
- [[Component testing with React Testing Library]]
- [[Developer experience (DX) MOC]]
