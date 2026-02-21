---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-locale-middleware"
---

# Redirecting to User's Preferred Locale

Use a proxy to detect the user's preferred language and redirect accordingly.

## Proxy Implementation

```typescript
// proxy.ts
import { NextRequest, NextResponse } from 'next/server';
import Negotiator from 'negotiator';
import { match } from '@formatjs/intl-localematcher';

const locales = ['en', 'fr', 'de'];
const defaultLocale = 'en';

function getLocale(request: NextRequest): string {
  const headers = { 'accept-language': request.headers.get('accept-language') || '' };
  const languages = new Negotiator({ headers }).languages();
  return match(languages, locales, defaultLocale);
}

export function proxy(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Check if pathname has a locale
  const pathnameHasLocale = locales.some(
    (locale) => pathname.startsWith(`/${locale}/`) || pathname === `/${locale}`
  );

  if (pathnameHasLocale) return;

  // Redirect to detected locale
  const locale = getLocale(request);
  request.nextUrl.pathname = `/${locale}${pathname}`;

  return NextResponse.redirect(request.nextUrl);
}

export default { fetch: proxy };
```

The proxy configuration is exported as a default object with a `fetch` handler. This detects the browser's language preference and redirects to the correct locale.

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*