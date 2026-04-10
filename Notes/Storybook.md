---
created: 2026-04-10
tags: [design-systems, storybook, documentation, testing]
related: [ [[Design systems MOC]], [[Design tokens and theming]], [[Component libraries]] ]
status: active
---

# Storybook

Storybook is the industry-standard tool for developing, documenting, and testing UI components **in isolation**, outside of the main application. Each component state is called a "story."

## What Storybook Solves

| Problem | Storybook Solution |
|---|---|
| Hard to test edge-case UI states | Each state is its own story |
| Components coupled to app routing/data | Stories run in isolation, no app context needed |
| No living documentation | Stories ARE the documentation |
| Designers can't review components without a running app | Storybook deploys as a static site |

---

## Setup (Next.js)

```bash
npx storybook@latest init
```

This auto-detects Next.js and installs the correct framework package.

---

## Writing Stories (Component Story Format 3)

```tsx
// components/Button/Button.stories.tsx
import type { Meta, StoryObj } from '@storybook/react';
import { Button } from './Button';

// Metadata for the component
const meta: Meta<typeof Button> = {
  title: 'UI/Button',       // Sidebar path in Storybook
  component: Button,
  tags: ['autodocs'],        // Auto-generate docs page from JSDoc + props
  args: {
    onClick: () => {},
  },
};
export default meta;

type Story = StoryObj<typeof Button>;

// Each named export is a story
export const Primary: Story = {
  args: {
    label: 'Click me',
    variant: 'primary',
  },
};

export const Secondary: Story = {
  args: {
    label: 'Cancel',
    variant: 'secondary',
  },
};

export const Disabled: Story = {
  args: {
    label: 'Submit',
    disabled: true,
  },
};

// Composing stories to test combinations
export const AllVariants: Story = {
  render: () => (
    <div style={{ display: 'flex', gap: 8 }}>
      <Button label="Primary" variant="primary" onClick={() => {}} />
      <Button label="Secondary" variant="secondary" onClick={() => {}} />
      <Button label="Ghost" variant="ghost" onClick={() => {}} />
    </div>
  ),
};
```

---

## Key Addons

| Addon | Purpose |
|---|---|
| `@storybook/addon-docs` | Auto-generates documentation from component + stories |
| `@storybook/addon-controls` | Interact with props live in the browser |
| `@storybook/addon-a11y` | Runs axe accessibility checks on every story |
| `@storybook/addon-viewport` | Preview at different screen sizes |
| `@storybook/addon-interactions` | Write and replay user interaction tests |
| `storybook-dark-mode` | Toggle dark mode in the Storybook canvas |

---

## Visual Regression Testing with Chromatic

**Chromatic** (made by the Storybook team) takes screenshots of every story on every PR and flags visual changes for review.

```bash
npm install -D chromatic
npx chromatic --project-token=<your-token>
```

Add to CI:
```yaml
- name: Chromatic visual tests
  run: npx chromatic --project-token=${{ secrets.CHROMATIC_PROJECT_TOKEN }}
```

Any PR that changes a component's appearance generates a visual diff — you approve or reject it. This prevents accidental UI regressions.

---

## Deploying Storybook

Storybook builds to a static site. Deploy it alongside your app so designers and PMs can always reference the latest components.

```bash
npm run build-storybook   # Outputs to storybook-static/
```

Tools: **Chromatic** (free tier), **Vercel**, **Netlify** — all support static site hosting.

---

## Related Topics
- [[Design systems MOC]]
- [[Design tokens and theming]]
- [[Component libraries]]
- [[Component testing with React Testing Library]]
- [[CI/CD for frontend]]
