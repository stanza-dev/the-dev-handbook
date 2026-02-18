---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-using-translations"
---

# Using Translations in Components

Pass the dictionary to your components for rendering translated content.

## In Server Components

```typescript
// app/[lang]/page.tsx
import { getDictionary } from '@/lib/translations';

export default async function Home({ params }: { params: { lang: string } }) {
  const dict = await getDictionary(params.lang);

  return (
    <div>
      <h1>{dict.home.title}</h1>
      <p>{dict.home.description}</p>
    </div>
  );
}
```

## In Client Components

Pass translations as props:

```typescript
// components/Header.tsx
'use client';

interface Props {
  translations: {
    home: string;
    about: string;
    contact: string;
  };
}

export function Header({ translations }: Props) {
  return (
    <nav>
      <a href="/">{translations.home}</a>
      <a href="/about">{translations.about}</a>
      <a href="/contact">{translations.contact}</a>
    </nav>
  );
}
```

## Interpolation

For dynamic values, use template strings or a library:

```json
{ "greeting": "Hello, {name}!" }
```

```typescript
dict.greeting.replace('{name}', user.name);
```

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*