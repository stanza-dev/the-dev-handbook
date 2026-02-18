---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-locale-metadata"
---

# Translating Metadata

Meta titles and descriptions should be translated for each locale.

## Dynamic Metadata

```typescript
// app/[lang]/page.tsx
import { getDictionary } from '@/lib/translations';
import type { Metadata } from 'next';

export async function generateMetadata({
  params,
}: {
  params: { lang: string };
}): Promise<Metadata> {
  const dict = await getDictionary(params.lang);

  return {
    title: dict.meta.homeTitle,
    description: dict.meta.homeDescription,
    openGraph: {
      title: dict.meta.homeTitle,
      description: dict.meta.homeDescription,
      locale: params.lang,
    },
  };
}
```

## Translation File

```json
// messages/en.json
{
  "meta": {
    "homeTitle": "Welcome | My Site",
    "homeDescription": "Build amazing products with Next.js"
  }
}

// messages/fr.json
{
  "meta": {
    "homeTitle": "Bienvenue | Mon Site",
    "homeDescription": "Créez des produits incroyables avec Next.js"
  }
}
```

## Sitemap

Include all localized URLs in your sitemap:

```typescript
// app/sitemap.ts
const locales = ['en', 'fr', 'de'];

export default function sitemap() {
  const routes = ['', '/about', '/contact'];

  return locales.flatMap((locale) =>
    routes.map((route) => ({
      url: `https://example.com/${locale}${route}`,
      lastModified: new Date(),
    }))
  );
}
```

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*