---
created: 2026-04-10
tags: [scalability, module-federation, micro-frontend, webpack]
related: [ [[Scalability MOC]], [[Monolith vs micro-frontend]], [[Monorepo architecture]] ]
status: active
---

# Module federation

**Module Federation** is a Webpack 5 feature that allows separately deployed JavaScript applications to share code **at runtime** — not at build time. It is the primary technical enabler of the **micro-frontend** architecture.

## The Core Problem It Solves

Without Module Federation, if three teams each build a separate app (`shell`, `checkout`, `account`), sharing React between them means:
- Either bundle React three times (wasteful, ~130kb × 3).
- Or go through a complex build-time linking process.

Module Federation allows the `checkout` app to say: "I need React. Let me use the React already loaded by the `shell` app."

---

## How It Works

There are two roles:

| Role | Description |
|---|---|
| **Remote** | An independently deployed app that **exposes** modules for others to consume |
| **Host** | An app that **consumes** modules from remotes at runtime |

An app can be both a host and a remote simultaneously.

---

## Webpack Configuration

```js
// checkout/webpack.config.js (Remote)
const { ModuleFederationPlugin } = require('webpack').container;

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'checkout',
      filename: 'remoteEntry.js',    // The manifest file for the host to load
      exposes: {
        './CheckoutPage': './src/CheckoutPage', // What we expose
      },
      shared: {
        react: { singleton: true, requiredVersion: '^18.0.0' },
        'react-dom': { singleton: true },
      },
    }),
  ],
};
```

```js
// shell/webpack.config.js (Host)
module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'shell',
      remotes: {
        // Point to the checkout team's deployed remoteEntry.js
        checkout: 'checkout@https://checkout.example.com/remoteEntry.js',
      },
      shared: { react: { singleton: true }, 'react-dom': { singleton: true } },
    }),
  ],
};
```

```tsx
// shell/src/App.tsx — Load the remote component lazily
import React, { lazy, Suspense } from 'react';
const CheckoutPage = lazy(() => import('checkout/CheckoutPage'));

export default function App() {
  return (
    <Suspense fallback={<div>Loading checkout...</div>}>
      <CheckoutPage />
    </Suspense>
  );
}
```

---

## Trade-offs

### Pros
- **True independent deployment**: Each team deploys on their own schedule.
- **No build-time coupling**: Shell doesn't need to rebuild when checkout updates.
- **Shared singletons**: React, design system components, and auth context can be shared globally.

### Cons
- **Versioning complexity**: Mismatched versions of shared libraries can cause runtime errors.
- **Testing difficulty**: Integration between remotes must be tested in an end-to-end environment.
- **Debugging**: Stack traces and error boundaries cross application boundaries.
- **Webpack lock-in**: Native Module Federation is Webpack-specific (Vite alternatives exist via plugins).

---

## When to Use It

✅ Large organization with multiple autonomous teams.
✅ Teams have genuinely different release cadences.
✅ The overhead of coordination between teams is measurable and painful.

❌ Small team — the complexity outweighs the benefits.
❌ Monorepo with shared build — use local packages instead.

---

## Related Topics
- [[Scalability MOC]]
- [[Monolith vs micro-frontend]]
- [[Monorepo architecture]]
- [[Route based code splitting]]
