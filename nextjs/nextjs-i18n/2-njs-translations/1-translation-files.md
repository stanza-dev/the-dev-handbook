---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-translation-files"
---

# Managing Translation Files

Translations are typically stored in JSON files organized by locale.

## File Structure

```
messages/
  en.json
  fr.json
  de.json
```

## Example Translation File

```json
// messages/en.json
{
  "common": {
    "home": "Home",
    "about": "About Us",
    "contact": "Contact"
  },
  "home": {
    "title": "Welcome to our site",
    "description": "We build amazing things"
  }
}
```

```json
// messages/fr.json
{
  "common": {
    "home": "Accueil",
    "about": "À propos",
    "contact": "Contact"
  },
  "home": {
    "title": "Bienvenue sur notre site",
    "description": "Nous créons des choses incroyables"
  }
}
```

## Loading Translations

```typescript
// lib/translations.ts
const dictionaries = {
  en: () => import('@/messages/en.json').then((m) => m.default),
  fr: () => import('@/messages/fr.json').then((m) => m.default),
};

export const getDictionary = async (locale: string) => {
  return dictionaries[locale]?.() ?? dictionaries.en();
};
```

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*