---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-formatting"
---

# Formatting Dates and Numbers

i18n libraries handle locale-specific formatting automatically.

## Date Formatting with next-intl

```typescript
import { useFormatter } from 'next-intl';

export function EventDate({ date }: { date: Date }) {
  const format = useFormatter();

  return (
    <time dateTime={date.toISOString()}>
      {format.dateTime(date, {
        year: 'numeric',
        month: 'long',
        day: 'numeric',
      })}
    </time>
  );
}

// en: December 25, 2024
// fr: 25 décembre 2024
// de: 25. Dezember 2024
```

## Number and Currency

```typescript
const format = useFormatter();

// Numbers
format.number(1234567.89); // 1,234,567.89 (en) / 1 234 567,89 (fr)

// Currency
format.number(99.99, { style: 'currency', currency: 'EUR' });
// €99.99 (en) / 99,99 € (fr)
```

## Relative Time

```typescript
format.relativeTime(new Date());
// "just now" / "à l'instant"

format.relativeTime(new Date(Date.now() - 86400000));
// "yesterday" / "hier"
```

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*