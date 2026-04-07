---
created: 2026-04-07
tags: [moc, i18n, internationalization]
related: [ [[Frontend system design MOC]] ]
status: active
---

# Internationalization (i18n)

Internationalization (i18n) is the process of designing and building your application so it can be adapted for different languages, regions, and cultures without code changes — only configuration changes.

> **i18n vs L10n**: i18n is the engineering work (making the app adaptable). Localization (l10n) is the translation work for a specific locale.

---

## What Needs to Be Internationalized?

| Concern | Example |
|---|---|
| **Text / UI Strings** | Buttons, labels, error messages |
| **Dates & Times** | `01/04/2024` vs `04/01/2024` vs `2024-04-01` |
| **Numbers & Currency** | `1,234.56` (US) vs `1.234,56` (DE) |
| **Pluralization** | "1 item" vs "2 items" (simple); Russian has 4 plural forms |
| **Text Direction** | LTR (English) vs RTL (Arabic, Hebrew) — see [[RTL support]] |
| **Images & Icons** | Some icons or flags may be culturally inappropriate |

---

## Notes in this Section
- [[RTL support]]: Implementing right-to-left layouts.
- [[Translation loading strategies]]: How and when to load translation files.

## Key Libraries

### `next-intl` (Next.js)
The recommended i18n library for the Next.js App Router.

```bash
npm install next-intl
```

### `react-i18next`
The most battle-tested i18n library for React, built on top of `i18next`.

---

## Core Concepts

### The `Intl` API
The browser's native internationalization API. No library needed for basic formatting:

```ts
// Date formatting
new Intl.DateTimeFormat('en-US').format(new Date()); // "4/7/2024"
new Intl.DateTimeFormat('de-DE').format(new Date()); // "7.4.2024"

// Number formatting
new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' }).format(1234.5);
// "$1,234.50"

// Plural rules
const pr = new Intl.PluralRules('en-US');
pr.select(1);  // "one"
pr.select(2);  // "other"
```

---
## Related Topics
- [[Frontend system design MOC]]
- [[Accessibility (a11y) MOC]]
- [[Rendering strategies MOC]] (SSR/SSG with locale-based routing)
