---
created: 2026-04-01
tags: [concept]
related: [ [[Requirement analysis in frontend design]], [[Functional vs non-functional requirements]], [[Accessibility (a11y) MOC]], [[Frontend system design MOC]] ]
status: active
---

# Accessibility requirements

## Overview

Accessibility (abbreviated **a11y** — 11 letters between 'a' and 'y') means designing your UI so that **everyone can use it**, including people with:
- Visual impairments (blindness, low vision, colour blindness)
- Motor impairments (cannot use a mouse — keyboard-only users)
- Cognitive impairments (difficulty processing complex layouts)
- Auditory impairments (for multimedia content)

Accessibility is not optional polish. In many countries (US, UK, EU), failure to meet accessibility standards is a **legal liability**.

> **Real example**: Domino's Pizza was sued in the US for an inaccessible website, and the US Supreme Court ruled the lawsuit could proceed. The settlement costs far exceeded the cost of making the site accessible from the start.

---

## The Standard: WCAG

The **Web Content Accessibility Guidelines (WCAG)** are the global standard for web accessibility, maintained by W3C.

### WCAG Conformance Levels

| Level | Description | When Required |
| :--- | :--- | :--- |
| **A** | Minimum. Removes the most critical barriers. | Bare minimum |
| **AA** | Standard. Covers most common disability needs. | **Target for most apps** |
| **AAA** | Maximum. Very strict, not always achievable for all content. | Government/specialist sites |

### The 4 Principles (POUR)

WCAG is organized around 4 core principles:

1. **Perceivable**: Information must be presentable to all users (e.g., text alternatives for images).
2. **Operable**: UI must be navigable by keyboard, not just mouse.
3. **Understandable**: Content must be readable and predictable.
4. **Robust**: Content must be interpreted by assistive technologies reliably.

---

## Key Accessibility Requirements You Need to Define

When gathering requirements, pin down which of these apply:

### 1. Semantic HTML
Are you using correct HTML elements for their purpose?

```html
<!-- Bad: Div soup -->
<div class="btn" onclick="submit()">Submit</div>

<!-- Good: Native semantics -->
<button type="submit">Submit</button>
```

A screen reader announces a `<button>` as clickable and keyboard-focusable by default. A `<div>` with an `onclick` is completely invisible to assistive technology.

### 2. Keyboard navigation
Every interactive element must be reachable and operable via keyboard alone (`Tab`, `Shift+Tab`, `Enter`, `Space`, arrow keys).

```typescript
// In Next.js / React — test this by pressing Tab
// All interactive elements need visible :focus styles
// Avoid outline: none unless you provide a custom focus indicator

// Example: custom focus ring
const Button = ({ children, onClick }) => (
  <button
    onClick={onClick}
    className="focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2"
  >
    {children}
  </button>
)
```

### 3. Colour contrast
WCAG AA requires:
- Normal text: **4.5:1** contrast ratio against background.
- Large text (18px+): **3:1** contrast ratio.
- UI components and icons: **3:1**.

```css
/* Bad: #999 text on white — ratio 2.85:1 (fails AA) */
color: #999;

/* Good: #767676 text on white — ratio 4.54:1 (passes AA) */
color: #767676;
```

Use the Chrome DevTools accessibility checker or [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/).

### 4. ARIA attributes
ARIA (Accessible Rich Internet Applications) attributes supplement HTML semantics when you must use custom UI patterns.

```typescript
// A custom modal needs ARIA roles for screen readers
export function Modal({ isOpen, onClose, title, children }) {
  return (
    <div
      role="dialog"
      aria-modal="true"
      aria-labelledby="modal-title"
      style={{ display: isOpen ? 'block' : 'none' }}
    >
      <h2 id="modal-title">{title}</h2>
      <div>{children}</div>
      <button onClick={onClose} aria-label="Close modal">×</button>
    </div>
  )
}
```

> **Rule**: Always prefer native HTML over ARIA. Only use ARIA when there is no native HTML element for the pattern (custom dropdown, slider, dialog).

### 5. Focus management
When a modal opens, focus should move into it. When it closes, focus should return to the trigger button.

```typescript
import { useEffect, useRef } from 'react'

export function Modal({ isOpen, onClose, triggerRef, children }) {
  const modalRef = useRef<HTMLDivElement>(null)

  useEffect(() => {
    if (isOpen) {
      // Move focus into modal when it opens
      modalRef.current?.focus()
    }
  }, [isOpen])

  const handleClose = () => {
    onClose()
    // Return focus to the button that opened the modal
    triggerRef.current?.focus()
  }

  return (
    <div ref={modalRef} tabIndex={-1} role="dialog">
      {children}
      <button onClick={handleClose}>Close</button>
    </div>
  )
}
```

---

## Tools for Testing Accessibility

| Tool | Type | Use for |
| :--- | :--- | :--- |
| **axe DevTools** (Chrome extension) | Automated | Quick audit of a11y violations |
| **Lighthouse** (Chrome DevTools) | Automated | Gives an a11y score |
| **NVDA** (Windows) | Screen reader | Manual testing with a real screen reader |
| **VoiceOver** (macOS/iOS) | Screen reader | Manual testing |
| **Tab key** | Manual | Check keyboard navigability |

---

## Accessibility in Next.js

Next.js has built-in a11y support:

```typescript
// next/image automatically generates alt-text prompt in TypeScript
import Image from 'next/image'

// TypeScript forces you to provide alt
<Image src="/hero.jpg" alt="Team working in an open office" width={800} height={400} />

// next/link renders a proper <a> tag — accessible by default
import Link from 'next/link'
<Link href="/dashboard">Go to Dashboard</Link>
```

Use the `eslint-plugin-jsx-a11y` plugin to catch a11y issues at development time:

```bash
npm install --save-dev eslint-plugin-jsx-a11y
```

```json
// .eslintrc.json
{
  "extends": ["next/core-web-vitals", "plugin:jsx-a11y/recommended"]
}
```

---

## Related Notes
- [[Accessibility (a11y) MOC]]
- [[Semantic HTML]]
- [[ARIA attributes]]
- [[Keyboard navigation and focus management]]
- [[Functional vs non-functional requirements]]
