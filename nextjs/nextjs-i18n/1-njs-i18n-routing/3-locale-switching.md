---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-locale-switching"
---

# Locale Switching

## Introduction

Users need to switch languages. Whether via dropdown, flags, or automatic detection, the switching mechanism must work smoothly without breaking navigation.

## Key Concepts

**Switching approaches**:

- URL-based: Change `/en/about` to `/fr/about`
- Cookie-based: Store preference for future visits
- User account: Save in database for logged-in users

## Deep Dive

### Locale Switcher Component

```typescript
// components/LocaleSwitcher.tsx
'use client';

import { usePathname, useRouter } from 'next/navigation';

const locales = ['en', 'fr', 'de'];

export function LocaleSwitcher({ currentLocale }: { currentLocale: string }) {
  const pathname = usePathname();
  const router = useRouter();

  const switchLocale = (newLocale: string) => {
    // Replace current locale in path
    const newPath = pathname.replace(`/${currentLocale}`, `/${newLocale}`);
    router.push(newPath);
  };

  return (
    <select
      value={currentLocale}
      onChange={(e) => switchLocale(e.target.value)}
    >
      {locales.map((locale) => (
        <option key={locale} value={locale}>
          {locale.toUpperCase()}
        </option>
      ))}
    </select>
  );
}
```

### Preserving Query Parameters

```typescript
const switchLocale = (newLocale: string) => {
  const url = new URL(window.location.href);
  const pathWithoutLocale = pathname.replace(`/${currentLocale}`, '');
  url.pathname = `/${newLocale}${pathWithoutLocale}`;
  router.push(url.toString());
};
```

### Saving Preference

```typescript
const switchLocale = (newLocale: string) => {
  // Save preference in cookie
  document.cookie = `NEXT_LOCALE=${newLocale}; path=/; max-age=31536000`;
  
  const newPath = pathname.replace(`/${currentLocale}`, `/${newLocale}`);
  router.push(newPath);
};
```

## Summary

Locale switching requires updating the URL path while preserving the current page. Save user preferences in cookies for returning visitors. Consider both dropdown and flag-based UI approaches.

## Resources

- [usePathname](https://nextjs.org/docs/app/api-reference/functions/use-pathname) — usePathname hook reference

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*