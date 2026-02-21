---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-locale-storage"
---

# Storing Locale Preference

## Introduction

Remembering a user's language preference improves UX. They shouldn't have to switch languages on every visit.

## Key Concepts

**Storage options**:

- Cookies: Work with SSR, read by middleware
- localStorage: Client-only, persists across sessions
- Database: For authenticated users

## Deep Dive

### Cookie-Based (Recommended)

```typescript
// proxy.ts
import { match } from '@formatjs/intl-localematcher';
import Negotiator from 'negotiator';

const locales = ['en', 'fr', 'de'];
const defaultLocale = 'en';

export function proxy(request: NextRequest) {
  // 1. Check cookie first
  const cookieLocale = request.cookies.get('NEXT_LOCALE')?.value;
  if (cookieLocale && locales.includes(cookieLocale)) {
    return redirectToLocale(request, cookieLocale);
  }

  // 2. Fall back to browser preference
  const browserLocale = getBrowserLocale(request);
  return redirectToLocale(request, browserLocale);
}

function getBrowserLocale(request: NextRequest): string {
  const headers = { 'accept-language': request.headers.get('accept-language') || '' };
  const languages = new Negotiator({ headers }).languages();
  return match(languages, locales, defaultLocale);
}
```

### Setting the Cookie

```typescript
// app/api/locale/route.ts
import { cookies } from 'next/headers';

export async function POST(request: Request) {
  const { locale } = await request.json();
  
  const cookieStore = await cookies();
  cookieStore.set('NEXT_LOCALE', locale, {
    maxAge: 60 * 60 * 24 * 365, // 1 year
    path: '/',
  });
  
  return Response.json({ success: true });
}
```

## Summary

Use cookies to store locale preference—they work with the proxy and SSR. Check cookie first, then fall back to browser Accept-Language header. Save preferences when users explicitly switch languages.

## Resources

- [Cookies](https://nextjs.org/docs/app/api-reference/functions/cookies) — Cookies API reference

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*