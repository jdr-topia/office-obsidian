---
created: 2026-04-02
tags: [concept, performance]
related: [ [[Performance optimization MOC]], [[Performance requirements]], [[Next.js Rendering Strategies (Complete Guide)]], [[Frontend system design MOC]] ]
status: active
---

# Code and bundle optimization

## Overview

Modern web applications often suffer from **JavaScript bloat**. Large JS bundles increase download times (especially on mobile/3G) and consume CPU time during parsing and execution.

Code and bundle optimization is about **reducing what you send** and ensuring it executes only when necessary.

---

## 1. Code Splitting (Next.js & React)

Instead of sending one massive `main.js` bundle, you split your application into smaller chunks that can be loaded on-demand.

### Route-Level Splitting (Automatic)
Next.js **automatically** splits your application by route. A user visiting `/dashboard` doesn't download the JavaScript for the `/settings` page until they navigate there.

### Component-Level Splitting (Dynamic Import)
Load heavy or conditional components only when they are needed.

```typescript
// app/dashboard/page.tsx
'use client'
import dynamic from 'next/dynamic'

// This component is only downloaded after clicking the button
const HeavyChart = dynamic(() => import('@/components/HeavyChart'), {
  ssr: false, // Don't render on server if it uses browser-only APIs
  loading: () => <p>Loading chartData...</p>,
})

export default function Dashboard() {
  const [showChart, setShowChart] = useState(false)

  return (
    <div>
      <button onClick={() => setShowChart(true)}>Show Sales Chart</button>
      {showChart && <HeavyChart />}
    </div>
  )
}
```

---

## 2. Tree Shaking

Tree shaking is the process of eliminating unused code (dead code) from your final bundle. Most modern bundlers (Webpack, Vite, Turbopack) do this by default using static analysis of `import` and `export` statements.

### How to Help Tree Shaking
- **Use ESM (ECMAScript Modules)**: Avoid using `module.exports` and `require()`.
- **Avoid Side Effects**: Ensure your functions are "pure" so the bundler can safely remove them if unused.
- **Selective Imports**: Instead of importing a whole library, import only what you need.

```typescript
// ❌ Potentially pulls in the entire library
import { debounce } from 'lodash';

// ✅ Pulls in only the debounce function
import debounce from 'lodash/debounce';
```

---

## 3. Bundle Analysis

You cannot optimize what you do not measure. Use a "Bundle Analyzer" to visualize your bundle composition.

### Tools: `next/bundle-analyzer`
Install and run it to see which libraries are taking up the most space.

```bash
# Install
npm install @next/bundle-analyzer

# Run analyzer during build
ANALYZE=true npm run build
```

**Common Findings from Bundle Analysis**:
- Outdated or large libraries (e.g., swapping `moment` for `date-fns`).
- Duplicate dependencies.
- Large CSS files included accidentally in the JS bundle.

---

## 4. Modern JavaScript (Polyfilling)

Modern browsers support most ES6+ features natively. Don't punish modern users with heavy polyfills for IE11.

- **Next.js Default**: Next.js target modern browsers by default, but you can customize this in `tsconfig.json` or through `browserslist`.
- **Target Modern**: Ensure your build target is set to a modern environment (e.g., `esnext` or `es2022`).

---

## 5. Third-Party Library Optimization

Third-party libraries are often the leading cause of bundle size issues.

- **Choose "Lightweight" Alternatives**: `date-fns` instead of `moment`, `lucide-react` instead of `font-awesome`.
- **Check for Modular Support**: Ensure the library supports tree shaking.
- **Avoid Heavy CSS Frameworks if not needed**: Use CSS Modules or Tailwind to keep CSS sizes low.

---

## Related Notes
- [[Performance optimization MOC]]
- [[Performance requirements]]
- [[Asset optimization in Next.js]]
- [[Frontend system design MOC]]
