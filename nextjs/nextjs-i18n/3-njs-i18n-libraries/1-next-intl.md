---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-next-intl"
---

# next-intl

`next-intl` is a popular i18n library designed specifically for Next.js.

## Installation

```bash
npm install next-intl
```

## Setup

```typescript
// i18n.ts
import { getRequestConfig } from 'next-intl/server';

export default getRequestConfig(async ({ locale }) => ({
  messages: (await import(`./messages/${locale}.json`)).default,
}));
```

## Middleware

```typescript
import createMiddleware from 'next-intl/middleware';

export default createMiddleware({
  locales: ['en', 'fr', 'de'],
  defaultLocale: 'en',
});
```

## Using in Components

```typescript
import { useTranslations } from 'next-intl';

export function Header() {
  const t = useTranslations('common');

  return (
    <nav>
      <a href="/">{t('home')}</a>
      <a href="/about">{t('about')}</a>
    </nav>
  );
}
```

## Features

- Automatic locale detection
- Type-safe translations
- Pluralization support
- Date/number formatting

## Resources

- [next-intl Documentation](https://next-intl-docs.vercel.app/) — Official next-intl documentation

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*