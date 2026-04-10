---
created: 2026-04-10
tags: [build, bundler, vite, webpack, turbopack]
related: [ [[Build and deployment MOC]], [[Code and bundle optimization]] ]
status: active
---

# Vite vs Webpack vs Turbopack

The bundler is the core of your frontend build system. It resolves module dependencies, applies transforms (TypeScript, JSX), and outputs optimized bundles. The choice has a massive impact on developer experience and build speed.

---

## Webpack

The veteran. Powers Create React App and many legacy setups.

### How It Works
Builds a complete dependency graph from entry points, applies loaders, and outputs bundles. Everything goes through a single JavaScript process.

### Pros
- **Ecosystem**: Massive plugin and loader ecosystem.
- **Battle-tested**: Used in production at scale for over a decade.
- **Flexible**: Can handle virtually any transformation pipeline.

### Cons
- **Slow cold starts**: Bundles everything upfront, even in dev mode.
- **Complex config**: `webpack.config.js` can become unwieldy.
- **No native ESM**: CommonJS-based; requires transforms.

---

## Vite

The current standard for new projects (powers Next.js dev mode, SvelteKit, Nuxt).

### How It Works
- **Development**: Uses **native browser ESM** — serves files as-is. No bundling in dev. The browser requests only what it needs.
- **Production**: Uses **Rollup** under the hood for optimized, tree-shaken bundles.

### Why It's Fast in Dev
```
Webpack: Scan entire app → build full bundle → serve
Vite:    Serve immediately → transform only requested files on demand
```

### Pros
- **Instant server start**: No pre-bundling of your app code.
- **HMR (Hot Module Replacement)**: Near-instant updates scoped to the exact changed module.
- **Simple config**: `vite.config.ts` is far simpler than Webpack.
- **ESM native**: Embraces modern JavaScript natively.

### Cons
- **Dev/prod parity**: Dev uses ESM, prod uses Rollup — subtle differences can cause prod-only bugs.
- **Large dependency pre-bundling**: `node_modules` are still pre-bundled by esbuild on first run.

---

## Turbopack

The new generation, built in Rust. Powers Next.js 13+ development server.

### How It Works
Uses **incremental computation** — only re-computes what has changed, caching the results of every build task at a granular level.

### Pros
- **Extremely fast HMR**: Benchmarks show 10×+ faster than Vite for large apps.
- **Rust-powered**: Native speed with no JavaScript runtime overhead.
- **Function-level caching**: Caches at the individual function level, not file level.

### Cons
- **Still maturing**: Not yet production-ready for all use cases.
- **Ecosystem**: Webpack plugin compatibility is still being built out.

---

## Comparison Summary

| | Webpack | Vite | Turbopack |
|---|---|---|---|
| **Language** | JavaScript | JavaScript + Rust (esbuild) | Rust |
| **Dev strategy** | Bundle everything | Native ESM, on-demand | Incremental, cached |
| **HMR speed** | Slow (seconds) | Fast (milliseconds) | Fastest (sub-millisecond) |
| **Config complexity** | High | Low | Low |
| **Production bundler** | Webpack | Rollup | Turbopack (upcoming) |
| **Best For** | Legacy/complex projects | New projects, SPAs | Next.js 13+ |

---

## Related Topics
- [[Build and deployment MOC]]
- [[Code and bundle optimization]]
- [[Route based code splitting]]
- [[CI/CD for frontend]]
