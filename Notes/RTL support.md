---
created: 2026-04-07
tags: [i18n, rtl, css, layout]
related: [ [[Internationalization (i18n)]], [[Accessibility (a11y) MOC]] ]
status: active
---

# RTL support

Right-to-left (RTL) support ensures your application is usable for languages like Arabic, Hebrew, Persian, and Urdu, where text flows from right to left and the entire layout is mirrored.

## The Layout Challenge

RTL is not just about flipping text. The entire UI is mirrored:
- Navigation: the sidebar moves from left → right.
- Breadcrumbs and steps flow right → left.
- Icons that imply direction (arrows, chevrons) are flipped.
- Scrollbars move to the left.
- Padding/margin on the left becomes padding/margin on the right.

---

## The Foundation: `dir` and `lang`

```html
<!-- Set at the HTML root level -->
<html lang="ar" dir="rtl">

<!-- Or on a specific element -->
<p dir="rtl" lang="ar">مرحبا بالعالم</p>
```

---

## CSS Logical Properties (The Modern Approach)

Old way: explicit physical directions (`left`, `right`, `margin-left`). These **do not flip** in RTL.

New way: **CSS Logical Properties** that map to physical directions based on writing mode.

| Physical | Logical Equivalent | LTR maps to | RTL maps to |
|---|---|---|---|
| `margin-left` | `margin-inline-start` | left | right |
| `margin-right` | `margin-inline-end` | right | left |
| `padding-left` | `padding-inline-start` | left | right |
| `border-left` | `border-inline-start` | left | right |
| `text-align: left` | `text-align: start` | left | right |

```css
/* ❌ Bad — breaks in RTL */
.card { padding-left: 16px; margin-right: 8px; }

/* ✅ Good — adapts automatically */
.card { padding-inline-start: 16px; margin-inline-end: 8px; }
```

---

## Handling in React / Next.js

### Dynamic `dir` Attribute
```tsx
// app/layout.tsx
import { getLocale } from 'next-intl/server';

const RTL_LOCALES = ['ar', 'he', 'fa', 'ur'];

export default async function RootLayout({ children }) {
  const locale = await getLocale();
  const dir = RTL_LOCALES.includes(locale) ? 'rtl' : 'ltr';

  return (
    <html lang={locale} dir={dir}>
      <body>{children}</body>
    </html>
  );
}
```

### Flipping Directional Icons
```tsx
const ChevronIcon = () => (
  <svg
    style={{ transform: 'var(--icon-flip, none)' }}
    /* ... */
  />
);

/* In CSS: */
[dir="rtl"] { --icon-flip: scaleX(-1); }
```

---

## Checklist for RTL Support

- [ ] Set `dir="rtl"` on `<html>` for RTL locales.
- [ ] Replace all physical CSS properties with logical equivalents.
- [ ] Mirror directional icons (arrows, chevrons, play buttons).
- [ ] Test text truncation — RTL text truncates on the left.
- [ ] Verify form input cursor starts on the right for RTL fields.
- [ ] Check animation/transition directions.

---

## Related Topics
- [[Internationalization (i18n)]]
- [[Translation loading strategies]]
- [[Accessibility (a11y) MOC]]
