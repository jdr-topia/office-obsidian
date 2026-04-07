---
created: 2026-04-07
tags: [observability, feature-flags, a-b-testing, experimentation]
related: [ [[Observability and monitoring MOC]], [[Performance monitoring]] ]
status: active
---

# Feature flags and A/B testing

Feature flags (also called feature toggles) decouple code deployment from feature release. A/B testing uses that same mechanism to run controlled experiments to make data-driven product decisions.

## Feature Flags

### What They Enable

| Use Case | Benefit |
|---|---|
| **Dark launches** | Deploy code months before release, test internally |
| **Gradual rollouts** | Release to 1%, 10%, 50%, then 100% of users |
| **Kill switches** | Instantly disable a broken feature without a redeploy |
| **Beta programs** | Enable features only for specific users or orgs |
| **Trunk-based development** | Merge incomplete features behind a flag |

---

### DIY Implementation (Simple)

For simple on/off flags, environment variables suffice:

```ts
// lib/flags.ts
export const flags = {
  newDashboard: process.env.NEXT_PUBLIC_FLAG_NEW_DASHBOARD === 'true',
  aiSearch: process.env.NEXT_PUBLIC_FLAG_AI_SEARCH === 'true',
};

// Usage in a component:
import { flags } from '@/lib/flags';

export default function Sidebar() {
  return (
    <nav>
      {flags.newDashboard ? <NewDashboardLink /> : <OldDashboardLink />}
    </nav>
  );
}
```

⚠️ **Limitation**: Changing a flag requires a redeploy. Use a dedicated service for dynamic flags.

---

### Production-Grade with a Flag Service

Services like **LaunchDarkly**, **Unleash**, **GrowthBook**, or **Vercel Edge Config** allow you to change flags in real-time without redeploying.

```ts
// Using Vercel Edge Config for dynamic flags
import { get } from '@vercel/edge-config';

export async function isFeatureEnabled(flag: string): Promise<boolean> {
  try {
    return (await get<boolean>(flag)) ?? false;
  } catch {
    return false; // Fail safely — default to disabled
  }
}

// In a Next.js Server Component or Route Handler:
const showNewCheckout = await isFeatureEnabled('new-checkout');
```

---

## A/B Testing

A/B testing is a controlled experiment comparing two variants (A = control, B = treatment) on a defined metric (conversion rate, click-through rate, etc.).

### Implementation Pattern

```ts
// 1. Assign a user to a cohort deterministically (same user always gets same variant)
function getVariant(userId: string, flagKey: string): 'control' | 'treatment' {
  const hash = simpleHash(`${userId}-${flagKey}`);
  return hash % 2 === 0 ? 'control' : 'treatment';
}

// 2. Render the correct variant
const variant = getVariant(session.user.id, 'checkout-cta-test');

return variant === 'treatment'
  ? <PrimaryCheckoutButton />
  : <DefaultCheckoutButton />;

// 3. Track the outcome
trackEvent('checkout_initiated', { variant, userId: session.user.id });
```

### With a Dedicated Tool (GrowthBook Example)

```tsx
import { useFeature } from '@growthbook/growthbook-react';

function CheckoutButton() {
  const { value: variant } = useFeature('checkout-cta-test');
  return variant === 'treatment'
    ? <PrimaryButton>Get Started Now</PrimaryButton>
    : <DefaultButton>Checkout</DefaultButton>;
}
```

---

## Flag Hygiene

> **A feature flag is technical debt.** Remove it once the rollout is complete.

- Add a **removal ticket** to your tracker when you create a flag.
- Set a **TTL** (time-to-live) expiry date on every flag.
- Conduct periodic **flag audits** to remove stale flags.

---

## Related Topics
- [[Observability and monitoring MOC]]
- [[Performance monitoring]]
- [[CI/CD for frontend]]
