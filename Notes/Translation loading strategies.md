---
created: 2026-04-07
tags: [i18n, performance, translations, next-intl]
related: [ [[Internationalization (i18n)]], [[Code and bundle optimization]], [[Route based code splitting]] ]
status: active
---

# Translation loading strategies

How and when translation files are loaded significantly impacts both bundle size and performance. The goal is to ship only the translations the current user actually needs.

---

## The Problem: Don't Ship All Translations at Once

An application with 20 languages and 500 keys per language has 10,000 translation strings. Loading all of them upfront is wasteful — a French user has no need for Japanese strings.

---

## Strategy 1: Locale-Based Route Splitting (Recommended)

Load only the translation bundle for the user's current locale. This pairs perfectly with server-side rendering.

### With `next-intl` (App Router)

```ts
// i18n.ts — Define supported locales
export const locales = ['en', 'ar', 'de', 'fr'] as const;
export const defaultLocale = 'en';
```

```ts
// messages/en.json
{
  "HomePage": {
    "title": "Welcome back",
    "description": "Here is what's new today."
  }
}

// messages/ar.json
{
  "HomePage": {
    "title": "مرحباً بعودتك",
    "description": "إليك ما هو جديد اليوم."
  }
}
```

```ts
// app/[locale]/layout.tsx
import { NextIntlClientProvider } from 'next-intl';
import { getMessages } from 'next-intl/server';

export default async function LocaleLayout({ children, params: { locale } }) {
  // Only the current locale's messages are loaded — server side
  const messages = await getMessages();

  return (
    <NextIntlClientProvider locale={locale} messages={messages}>
      {children}
    </NextIntlClientProvider>
  );
}
```

---

## Strategy 2: Namespace Splitting

For very large apps, split translations by feature/page namespace and load them on demand.

```ts
// Load only the "checkout" namespace when the user visits /checkout
const { t } = useTranslation('checkout');

// react-i18next lazy loads on first use:
i18n.loadNamespaces('checkout');
```

---

## Strategy 3: Dynamic Import at Runtime

For client-rendered apps (SPAs) that switch locale without a page reload:

```ts
import i18n from 'i18next';

async function changeLanguage(locale: string) {
  if (!i18n.hasResourceBundle(locale, 'translation')) {
    // Load the translation file dynamically only when needed
    const messages = await import(`../locales/${locale}/translation.json`);
    i18n.addResourceBundle(locale, 'translation', messages.default);
  }
  await i18n.changeLanguage(locale);
}
```

---

## Translation File Best Practices

1. **Use nested keys** to group by feature: `{ "auth": { "login": "Log in", "logout": "Log out" } }`
2. **Use ICU message format** for pluralization and interpolation:
   ```json
   { "items": "{count, plural, one {# item} other {# items}}" }
   ```
3. **String IDs, not sentences**: Use keys like `common.submit` not `"Submit"` — avoids key collisions.
4. **Translation Management Systems (TMS)**: For teams, use Crowdin, Phrase, or Localazy to manage translation workflows.

---

## Related Topics
- [[Internationalization (i18n)]]
- [[RTL support]]
- [[Route based code splitting]]
- [[Code and bundle optimization]]
