---
name: Feature Module Scaffolding
description: Scaffolds a new feature module following the project's atomic design and Next.js 16 structure.
---

# Feature Module Scaffolding Skill

This skill helps you create a structured feature module within the `campaign_manager` project. It ensures consistency with the existing architecture.

## Usage

When the user asks to "create a new feature" or "add a module", follow these steps.

## Steps

1.  **Identify Feature Name**: Determining the feature name (e.g., `campaigns`, `contacts`, `analytics`). Use plural nouns.
2.  **Create Directory Structure**:
    - `app/(main)/[feature-name]/`
    - `app/(main)/[feature-name]/components/`
    - `app/(main)/[feature-name]/hooks/`
    - `app/(main)/[feature-name]/services/`
3.  **Create Entry Point**:
    - Create `app/(main)/[feature-name]/page.tsx`.
    - This should be a server component (default in App Router) or client component `"use client"` if needed.
    - Export a default function named `[FeatureName]Page`.
4.  **Create Layout (Optional)**:
    - If the feature needs a specific layout, create `app/(main)/[feature-name]/layout.tsx`.
5.  **Create Type Definition (Optional)**:
    - If the feature introduces new domain entities, create `app/(main)/[feature-name]/[feature-name].types.ts` OR update `types/index.ts`.
6.  **Register Navigation**:
    - If the feature needs to appear in the Sidebar:
      - Open `shared/constants/AppConst.ts` (or standard constant file).
      - Add a new menu item entry.

## Example Output Structure

```
app/(main)/analytics/
├── components/
│   └── AnalyticsChart.tsx
├── hooks/
│   └── useAnalyticsData.ts
├── services/
│   └── analytics.service.ts
├── page.tsx
└── layout.tsx
```
