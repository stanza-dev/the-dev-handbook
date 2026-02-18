---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-hreflang"
---

# Hreflang for SEO

Hreflang tags tell search engines which language versions of a page exist.

## Adding Hreflang in Metadata

```typescript
// app/[lang]/page.tsx
import type { Metadata } from 'next';

const locales = ['en', 'fr', 'de'];

export async function generateMetadata({
  params,
}: {
  params: { lang: string };
}): Promise<Metadata> {
  return {
    alternates: {
      canonical: `https://example.com/${params.lang}`,
      languages: Object.fromEntries(
        locales.map((locale) => [locale, `https://example.com/${locale}`])
      ),
    },
  };
}
```

## Generated Output

```html
<link rel="alternate" hreflang="en" href="https://example.com/en" />
<link rel="alternate" hreflang="fr" href="https://example.com/fr" />
<link rel="alternate" hreflang="de" href="https://example.com/de" />
<link rel="canonical" href="https://example.com/en" />
```

## x-default

For the fallback language:

```typescript
alternates: {
  languages: {
    'x-default': 'https://example.com/en',
    en: 'https://example.com/en',
    fr: 'https://example.com/fr',
  },
}
```

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*