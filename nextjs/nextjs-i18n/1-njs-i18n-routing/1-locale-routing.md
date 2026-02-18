---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-locale-routing"
---

# Locale-Based Routing

Next.js supports internationalized routing natively using middleware and route segments.

## Strategy: URL Segments

The most common pattern is including the locale in the URL:

- `/en/about` → English
- `/fr/about` → French
- `/de/about` → German

## Folder Structure

```
app/
  [lang]/
    page.tsx
    about/
      page.tsx
    layout.tsx
```

The `[lang]` segment captures the locale.

## Accessing the Locale

```typescript
// app/[lang]/page.tsx
export default function Home({ params }: { params: { lang: string } }) {
  return <h1>Current locale: {params.lang}</h1>;
}

// Generate static params for all locales
export async function generateStaticParams() {
  return [{ lang: 'en' }, { lang: 'fr' }, { lang: 'de' }];
}
```

## Resources

- [Internationalization](https://nextjs.org/docs/app/building-your-application/routing/internationalization) — Official i18n routing guide

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*