---
name: ui-from-reference-image
description: Helps with generating pixel-perfect UI from reference images while maintaining project consistency.
---

# UI Generation from Reference Image Skill

When generating UI from a reference image, follow this comprehensive workflow to ensure pixel-perfect results that align with the project's technical architecture.

## 1. Tech Stack Alignment

Always prioritize project-specific libraries and patterns:

- **Core:** Next.js (App Router), TypeScript.
- **UI Framework:** [Ant Design (antd)](https://ant.design/components/overview/). Use Antd components (Table, Modal, Drawer, Form, etc.) wherever possible.
- **Styling:** [Tailwind CSS](https://tailwindcss.com/) for layout, spacing, and custom styling. **Prioritize using Tailwind utility classes and project theme tokens over raw hex codes.**
- **Icons:** [Lucide React](https://lucide.dev/icons/) or Ant Design Icons.
- **Animations:** [Framer Motion](https://www.framer.com/motion/) for smooth transitions.
- **State Management:** [Zustand](https://docs.pmnd.rs/zustand/getting-started/introduction).

## 2. Review Checklist (Discovery Phase)

1.  **Analyze Visuals:** Break down the design into layout (Flex/Grid), typography (Poppins), and color palette. If Dark mode reference design is absent, generating compelling dark mode theme that aligns with existing pages.
2.  **Color Consistency:** Check the [RI Updates Color Scheme](file:///c:/JD/projects/live/SafePhv-RI-Design/.agent/skills/ui-from-reference-image/resources/color_scheme.md) to ensure correct hex codes for Light and Dark modes.
3.  **Identify Components:** List Ant Design components that can be used (e.g., `Card`, `Tag`, `Typography`).
4.  **Find Reusables:** Check the `components/` and `shared/` directories for existing project-specific components.
5.  **Clarify:** Ask the user about specific behaviors (e.g., "Is this card clickable?", "Does this drawer close on overlay click?").

## 3. Workflow

### Phase A: Detailed Planning

- Document the proposed structure in `implementation_plan.md`.
- Map UI elements to specific code files and Antd components.
- Define the data structure/props needed for the UI.

### Phase B: Clean Execution

- Implement the layout foundation using Tailwind CSS.
- Layer Ant Design components for functionality and standard UI elements.
- Apply custom CSS/Tailwind for fine-tuning to match the reference image.
- **CRITICAL:** Do not break existing code. Avoid unnecessary refactors of stable components.

### Phase C: Assets & Micro-interactions

- Use the `generate_image` tool if placeholder assets are needed.
- Implement subtle hover effects and entry animations using Framer Motion.

## 4. Verification Patterns

- **Responsive Check:** Verify the UI works across different screen sizes.
- **Accessibility:** Ensure buttons have labels and components are keyboard-navigable.

> [!IMPORTANT]
> Always maintain the project's coding standards: no reformatting, keep existing indentation, and follow the established folder structure.
