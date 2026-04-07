---
created: 2026-04-07
tags: [accessibility, aria, a11y]
related: [ [[Accessibility (a11y) MOC]], [[Semantic HTML]], [[Keyboard navigation and focus management]] ]
status: active
---

# ARIA attributes

**ARIA (Accessible Rich Internet Applications)** is a set of HTML attributes that extend the semantics of elements, providing information to assistive technologies when native HTML semantics are insufficient.

## The First Rule of ARIA

> **Do not use ARIA if you can use a native HTML element instead.**

ARIA communicates intent to screen readers but does not add behavior. A `role="button"` on a `<div>` does not give it keyboard focus or click-on-enter — you must add that yourself. A `<button>` gets all of this for free.

---

## The Three Categories of ARIA

### 1. `role` — What is this element?
Overrides or defines the element's role in the accessibility tree.

```html
<div role="dialog" aria-modal="true" aria-labelledby="dialog-title">
  <h2 id="dialog-title">Confirm Deletion</h2>
</div>

<ul role="listbox">
  <li role="option" aria-selected="true">Option A</li>
  <li role="option" aria-selected="false">Option B</li>
</ul>
```

### 2. `aria-*` Properties — What are its characteristics?
Static information describing an element.

| Attribute | Purpose |
|---|---|
| `aria-label="Close"` | Provides an accessible name (when no visible text exists) |
| `aria-labelledby="id"` | Points to another element whose text is the label |
| `aria-describedby="id"` | Points to a longer description element |
| `aria-required="true"` | Marks a form field as required |
| `aria-invalid="true"` | Marks a field as having an error |
| `aria-haspopup="listbox"` | Indicates the element triggers a popup |

### 3. `aria-*` States — What is its current state?
Dynamic information that changes with user interaction.

| Attribute | Purpose |
|---|---|
| `aria-expanded="true/false"` | Whether a collapsible section is open |
| `aria-hidden="true"` | Hides the element from the accessibility tree (decorative icons) |
| `aria-selected="true/false"` | Which item in a list is selected |
| `aria-checked="true/false"` | Checkbox or toggle state |
| `aria-busy="true"` | Element is loading/updating |
| `aria-live="polite"` | Announces dynamic content changes to screen readers |

---

## Common Patterns in React

### Accessible Icon Button
```tsx
// A button with only an icon needs an aria-label
<button aria-label="Close dialog" onClick={onClose}>
  <XIcon aria-hidden="true" /> {/* Hide icon from screen reader */}
</button>
```

### Live Regions (Dynamic Announcements)
```tsx
// Screen reader will announce changes to this element
function StatusMessage({ message, isError }) {
  return (
    <div
      role="status"            // For non-urgent updates
      aria-live="polite"       // Wait for user to finish current task
      aria-atomic="true"       // Read the entire region on update
    >
      {message}
    </div>
  );
}

// For urgent alerts (e.g., form errors):
<div role="alert" aria-live="assertive">{errorMessage}</div>
```

### Accessible Accordion
```tsx
function AccordionItem({ title, content, isOpen, onToggle }) {
  const contentId = `content-${title}`;
  return (
    <div>
      <button
        aria-expanded={isOpen}
        aria-controls={contentId}
        onClick={onToggle}
      >
        {title}
      </button>
      <div id={contentId} hidden={!isOpen}>
        {content}
      </div>
    </div>
  );
}
```

---

## Related Topics
- [[Accessibility (a11y) MOC]]
- [[Semantic HTML]]
- [[Keyboard navigation and focus management]]
- [[Headless components]] (Radix UI handles ARIA automatically)
