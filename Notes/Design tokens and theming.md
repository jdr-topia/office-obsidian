---
created: 2026-04-10
tags: [design-systems, design-tokens, theming, css-variables]
related: [ [[Design systems MOC]], [[Component libraries]], [[Storybook]] ]
status: active
---

# Design tokens and theming

**Design tokens** are the named, technology-agnostic variables that represent every visual decision in your design system — colours, spacing, typography, border radii, and shadows. They are the bridge between design tools (Figma) and code (CSS, React).

> A design token is not just a CSS variable. It is a **decision** captured in a format that can be consumed by any platform.

---

## Why Tokens Matter

Without tokens:
- A designer changes the brand blue → developer hunts through hundreds of files.
- Light/dark mode requires duplicating every colour value.
- Web, iOS, and Android each have their own hard-coded values with no single source of truth.

With tokens:
- Change `--color-primary` in one place → the entire product updates.
- Dark mode is a token swap, not a code rewrite.

---

## Token Categories

| Category | Example Token | Example Value |
|---|---|---|
| **Color** | `color.brand.primary` | `#4A90E2` |
| **Spacing** | `spacing.4` | `16px` |
| **Typography** | `font.size.lg` | `18px` |
| **Border Radius** | `radius.md` | `8px` |
| **Shadow** | `shadow.card` | `0 2px 8px rgba(0,0,0,.12)` |
| **Duration** | `duration.fast` | `150ms` |

---

## Implementation with CSS Custom Properties

CSS variables are the most universally compatible way to implement tokens on the web.

```css
/* tokens.css — the source of truth */
:root {
  /* Colour palette (primitive tokens) */
  --color-blue-500: #4A90E2;
  --color-blue-700: #2C6DB5;
  --color-neutral-900: #1A1A2E;
  --color-neutral-100: #F5F5F5;

  /* Semantic tokens — map primitives to intent */
  --color-primary: var(--color-blue-500);
  --color-primary-hover: var(--color-blue-700);
  --color-bg-surface: var(--color-neutral-100);
  --color-text-default: var(--color-neutral-900);

  /* Spacing scale */
  --spacing-1: 4px;
  --spacing-2: 8px;
  --spacing-4: 16px;
  --spacing-8: 32px;

  /* Typography */
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
  --font-size-lg: 1.125rem;
  --font-weight-medium: 500;
  --font-weight-bold: 700;

  /* Radii */
  --radius-sm: 4px;
  --radius-md: 8px;
  --radius-lg: 16px;
}
```

---

## Dark Mode Theming

Swap semantic tokens using a data attribute or media query — no component code changes needed.

```css
/* Light mode (default) */
:root {
  --color-bg-surface: #ffffff;
  --color-text-default: #1a1a2e;
  --color-border: #e5e7eb;
}

/* Dark mode — just override the semantic tokens */
[data-theme="dark"] {
  --color-bg-surface: #0f172a;
  --color-text-default: #f1f5f9;
  --color-border: #1e293b;
}

/* Or use the OS preference */
@media (prefers-color-scheme: dark) {
  :root {
    --color-bg-surface: #0f172a;
    --color-text-default: #f1f5f9;
  }
}
```

```tsx
// ThemeToggle.tsx
export function ThemeToggle() {
  const toggleTheme = () => {
    const current = document.documentElement.dataset.theme;
    document.documentElement.dataset.theme = current === 'dark' ? 'light' : 'dark';
  };
  return <button onClick={toggleTheme}>Toggle Theme</button>;
}
```

---

## Token Tooling

| Tool | Purpose |
|---|---|
| **Style Dictionary** (Amazon) | Transform tokens (JSON) → CSS, iOS, Android formats |
| **Theo** (Salesforce) | Token transformation pipeline |
| **Figma Tokens plugin** | Sync tokens bidirectionally between Figma and code |
| **CSS Custom Properties** | Native browser token implementation |

---

## Related Topics
- [[Design systems MOC]]
- [[Component libraries]]
- [[Storybook]]
- [[Accessibility (a11y) MOC]] (colour contrast is a token-level decision)
