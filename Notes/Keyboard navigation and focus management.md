---
created: 2026-04-07
tags: [accessibility, keyboard, focus, a11y]
related: [ [[Accessibility (a11y) MOC]], [[ARIA attributes]], [[Semantic HTML]] ]
status: active
---

# Keyboard navigation and focus management

Keyboard accessibility ensures users can interact with every part of your application using only a keyboard. This is critical for motor-impaired users and power users who prefer the keyboard.

## The Core Requirement

> Every interactive element that is accessible by mouse must also be accessible by keyboard.

---

## Tab Order & `tabIndex`

By default, the browser moves focus through `<a>`, `<button>`, `<input>`, `<select>`, and `<textarea>` in DOM order when the user presses `Tab`.

### `tabIndex` Values
| Value | Behavior |
|---|---|
| `tabIndex={0}` | Adds the element to the natural tab order |
| `tabIndex={-1}` | Removes from tab order but allows JS `.focus()` |
| `tabIndex={1+}` | Forces a specific order — **avoid this**, it creates confusing behavior |

```tsx
// Make a custom element keyboard focusable
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => e.key === 'Enter' && handleClick()}
>
  Custom Button
</div>

// Much better — just use a real button:
<button onClick={handleClick}>Custom Button</button>
```

---

## Focus Management in Modals

When a modal opens, focus must move into it. When it closes, focus must return to the trigger element.

```tsx
import { useEffect, useRef } from 'react';

function Modal({ isOpen, onClose, children }) {
  const modalRef = useRef<HTMLDivElement>(null);
  const triggerRef = useRef<HTMLButtonElement>(null);

  useEffect(() => {
    if (isOpen) {
      // Move focus to the modal when it opens
      modalRef.current?.focus();
    } else {
      // Return focus to the trigger when it closes
      triggerRef.current?.focus();
    }
  }, [isOpen]);

  if (!isOpen) return null;

  return (
    <div
      ref={modalRef}
      role="dialog"
      aria-modal="true"
      tabIndex={-1}
    >
      {children}
      <button onClick={onClose}>Close</button>
    </div>
  );
}
```

---

## Focus Trapping

When a modal or drawer is open, focus should be **trapped inside** it. Users pressing `Tab` should cycle through elements within the modal only.

```tsx
// Use the 'focus-trap-react' library for a robust solution
import FocusTrap from 'focus-trap-react';

function Modal({ isOpen, onClose }) {
  return isOpen ? (
    <FocusTrap>
      <div role="dialog" aria-modal="true">
        <h2>Dialog Title</h2>
        <button>Action</button>
        <button onClick={onClose}>Close</button>
      </div>
    </FocusTrap>
  ) : null;
}
```

---

## Visible Focus Indicators

Never suppress focus outlines without providing an equivalent visual alternative:

```css
/* ❌ Bad — completely removes focus visibility */
:focus { outline: none; }

/* ✅ Good — replace the default with a better-looking one */
:focus-visible {
  outline: 2px solid #4A90E2;
  outline-offset: 3px;
  border-radius: 4px;
}

/* :focus-visible only shows the ring for keyboard navigation,
   not for mouse clicks — best of both worlds */
```

---

## Key Keyboard Event Patterns

| Component | Keys to Handle |
|---|---|
| Button / Custom control | `Enter`, `Space` |
| Dialog / Modal | `Escape` to close |
| Dropdown / Listbox | `ArrowUp`, `ArrowDown`, `Enter`, `Escape` |
| Tab panel | `ArrowLeft`, `ArrowRight` between tabs |
| Date picker | Full arrow key grid navigation |

---

## Related Topics
- [[Accessibility (a11y) MOC]]
- [[ARIA attributes]]
- [[Semantic HTML]]
- [[Headless components]] (headless libs handle all keyboard patterns for you)
