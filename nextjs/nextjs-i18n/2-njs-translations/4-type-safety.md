---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-type-safety"
---

# Type-Safe Translations

## Introduction

Typos in translation keys are silent bugs—your app shows the key instead of the translated text. TypeScript can catch these at compile time.

## Key Concepts

**Type safety benefits**:

- Autocomplete for translation keys
- Compile-time errors for missing keys
- Refactoring support

## Deep Dive

### Generating Types from JSON

```typescript
// lib/translations.ts
import en from '@/messages/en.json';

export type Messages = typeof en;

export function getDictionary(locale: string): Promise<Messages> {
  // ... 
}
```

### Type-Safe Access

```typescript
// Using the type
function getNestedValue<T extends Messages>(
  dict: T,
  path: string
): string {
  return path.split('.').reduce((obj, key) => obj?.[key], dict as any);
}

// With autocomplete
dict.home.title; // TypeScript knows this exists
dict.home.typo;  // Error: Property 'typo' does not exist
```

### next-intl Type Safety

```typescript
// global.d.ts
type Messages = typeof import('@/messages/en.json');

declare interface IntlMessages extends Messages {}
```

```typescript
// Usage - fully typed
const t = useTranslations('home');
t('title');  // Autocomplete works
t('typo');   // TypeScript error
```

## Summary

Generate types from your base translation file to catch typos at compile time. Both manual approaches and next-intl support full type safety with autocomplete.

## Resources

- [next-intl TypeScript](https://next-intl-docs.vercel.app/docs/workflows/typescript) — Type-safe translations with next-intl

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*