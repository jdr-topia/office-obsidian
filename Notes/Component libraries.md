---
created: 2026-04-10
tags: [design-systems, component-library, radix, shadcn]
related: [ [[Design systems MOC]], [[Headless components]], [[Atomic design pattern]] ]
status: active
---

# Component libraries

A component library is a collection of pre-built, reusable UI components. The key architectural decision is: **build your own** vs **adopt an existing library** vs **use headless primitives and style them yourself**.

---

## The Spectrum of Component Libraries

```
Less flexible ◄─────────────────────────────────► More flexible
Styled libraries → Unstyled/Headless → Build your own
(MUI, Ant Design)   (Radix, Headless UI)   (from scratch)
```

---

## Category 1: Fully Styled Libraries

Include their own CSS/styling. You get consistent, production-ready components quickly.

### Material UI (MUI)
- Based on Google's Material Design.
- `sx` prop and `styled()` API for customization.
- Strong TypeScript support, large ecosystem.

```tsx
import { Button, TextField, Stack } from '@mui/material';

<Stack spacing={2}>
  <TextField label="Email" variant="outlined" />
  <Button variant="contained" color="primary">Submit</Button>
</Stack>
```

### When to use: Rapid prototyping, internal tools, when brand customization is minimal.

---

## Category 2: Headless / Unstyled Libraries

Provide logic, accessibility, and state — zero styles. You bring your own CSS.

### Radix UI
Each primitive is installable independently. Provides ARIA, keyboard nav, and focus management out of the box.

```bash
npm install @radix-ui/react-dialog
```

```tsx
import * as Dialog from '@radix-ui/react-dialog';

<Dialog.Root>
  <Dialog.Trigger asChild>
    <button>Open</button>
  </Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay className="dialog-overlay" />
    <Dialog.Content className="dialog-content">
      <Dialog.Title>Are you sure?</Dialog.Title>
      <Dialog.Close>Cancel</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

### When to use: When you need full visual control and have a custom design system.

---

## Category 3: shadcn/ui (The Modern Hybrid)

Not a library you install — it's a **collection of copy-pasteable components** built on Radix UI + Tailwind CSS. You own the code.

```bash
npx shadcn@latest add button dialog input
```

This copies the component source into your project at `components/ui/`. You can modify it freely.

**Why it's popular:**
- Full ownership — no library upgrade breakage.
- Built on Radix (great a11y).
- Styled with Tailwind (easy customization).
- Excellent TypeScript support.

---

## Decision Framework

| Question | Lean towards |
|---|---|
| Do we have a brand design system? | Headless (Radix, shadcn) |
| Is this an internal/admin tool? | Fully styled (MUI, Ant Design) |
| Do we need maximum a11y? | Headless (a11y is built in) |
| Small team, fast delivery? | Fully styled or shadcn |
| Customer-facing product with unique branding? | Headless or custom |

---

## Related Topics
- [[Design systems MOC]]
- [[Headless components]]
- [[Design tokens and theming]]
- [[Atomic design pattern]]
