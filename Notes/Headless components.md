---
created: 2026-04-02
tags: [ui-architecture, react-patterns]
related: [ [[UI architecture MOC]], [[Atomic design pattern]] ]
status: active
---

# Headless components

A **headless component** is a component that provides logic, state, and accessibility features without providing any markup or styling. It leaves the "visual representation" (the head) entirely to the consumer.

## The Problem Solved

Traditional UI libraries (like Bootstrap or older MUI) combine logic with styling. If you want to change the look of a `Select` component but keep the keyboard navigation and state logic, you are often blocked by the library's internal CSS.

## How it works

A headless component uses patterns like **Hooks** or **Render Props** to expose internal state and event handlers.

### Example: TanStack Table (Headless)
Instead of providing a `<Table />` tag, it provides a `useTable` hook that calculates header positions, sorting state, and pagination logic. You then apply those results to your own `<table>`, `<div>`, or `flexbox` structure.

### Example: Radix UI / Headless UI
These libraries provide the "behavior" for complex components like Modals, Tooltips, and Tabs.

```tsx
import * as Accordion from '@radix-ui/react-accordion';

// You provide YOUR styles (using Tailwind, CSS Modules, or anything)
const MyAccordion = () => (
  <Accordion.Root className="AccordionRoot">
    <Accordion.Item value="item-1">
      <Accordion.Header className="AccordionHeader">
        <Accordion.Trigger className="AccordionTrigger">
          Trigger content
        </Accordion.Trigger>
      </Accordion.Header>
      <Accordion.Content className="AccordionContent">
        Panel content
      </Accordion.Content>
    </Accordion.Item>
  </Accordion.Root>
);
```

---

## Why Use Headless?

1.  **Style Freedom**: Complete control over CSS/Tailwind/Framer Motion.
2.  **Accessibility (a11y)**: Logic for ARIA attributes, keyboard traps, and focus management is built-in.
3.  **Separation of Concerns**: Logic is decoupled from the UI.
4.  **Testability**: You can test the logic independently of the DOM structure.

---
## Related Patterns
- [[Compound components]]
- [[UI architecture MOC]]
- [[Accessibility (a11y) MOC]]
