---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-og-images"
---

# Localized Open Graph Images

## Introduction

Social media previews should be in the user's language. A French user sharing your link should see a French preview image, not English.

## Key Concepts

**OG image localization**:

- Static: Pre-generated images per locale
- Dynamic: Generated on-the-fly with locale text
- Hybrid: Template image + dynamic text overlay

## Deep Dive

### Locale-Specific Static Images

```typescript
// app/[lang]/page.tsx
export async function generateMetadata({ 
  params 
}: { 
  params: { lang: string } 
}): Promise<Metadata> {
  return {
    openGraph: {
      images: [`/og/${params.lang}/home.png`],
    },
  };
}
```

### Dynamic OG Images

```typescript
// app/[lang]/og/route.tsx
import { ImageResponse } from 'next/og';
import { getDictionary } from '@/lib/translations';

export async function GET(request: Request) {
  const { searchParams } = new URL(request.url);
  const lang = searchParams.get('lang') || 'en';
  
  const dict = await getDictionary(lang);

  return new ImageResponse(
    (
      <div
        style={{
          fontSize: 72,
          background: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
          width: '100%',
          height: '100%',
          display: 'flex',
          alignItems: 'center',
          justifyContent: 'center',
          color: 'white',
        }}
      >
        {dict.meta.siteTitle}
      </div>
    ),
    { width: 1200, height: 630 }
  );
}
```

### Using in Metadata

```typescript
export async function generateMetadata({ params }): Promise<Metadata> {
  return {
    openGraph: {
      images: [
        {
          url: `/api/og?lang=${params.lang}&title=${encodeURIComponent(title)}`,
          width: 1200,
          height: 630,
          alt: title,
        },
      ],
    },
  };
}
```

## Summary

Localize OG images for better social engagement. Use static images for main pages, dynamic ImageResponse for article-specific images. Always include localized alt text.

## Resources

- [ImageResponse](https://nextjs.org/docs/app/api-reference/functions/image-response) — Dynamic OG image generation

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*