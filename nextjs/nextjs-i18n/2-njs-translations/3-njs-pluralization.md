---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-pluralization"
---

# Pluralization and Variables

## Introduction

Real translations need dynamic content: "You have 1 item" vs "You have 5 items." Pluralization rules vary wildly between languages—English has 2 forms, Arabic has 6.

## Key Concepts

**Dynamic translation features**:

- Variable interpolation: `Hello, {name}!`
- Pluralization: Different forms based on count
- Gender agreement: He/She variations

## Deep Dive

### Simple Interpolation

```json
// messages/en.json
{
  "greeting": "Hello, {name}!",
  "items": "You have {count} items in your cart."
}
```

```typescript
function replaceVariables(template: string, values: Record<string, string | number>) {
  return template.replace(/{(\w+)}/g, (_, key) => String(values[key] || ''));
}

replaceVariables(dict.greeting, { name: 'John' });
// "Hello, John!"
```

### ICU Message Format (with next-intl)

```json
// messages/en.json
{
  "cartItems": "{count, plural, =0 {Your cart is empty} one {You have 1 item} other {You have {count} items}}"
}
```

```typescript
// With next-intl
import { useTranslations } from 'next-intl';

function Cart({ itemCount }: { itemCount: number }) {
  const t = useTranslations();
  return <p>{t('cartItems', { count: itemCount })}</p>;
}
// itemCount=0: "Your cart is empty"
// itemCount=1: "You have 1 item"
// itemCount=5: "You have 5 items"
```

### Complex Pluralization (Arabic)

```json
// messages/ar.json
{
  "items": "{count, plural, =0 {لا توجد عناصر} one {عنصر واحد} two {عنصران} few {{count} عناصر} many {{count} عنصرًا} other {{count} عنصر}}"
}
```

## Summary

Use ICU Message Format for complex pluralization—it handles the linguistic rules automatically. For simple interpolation, a basic replace function works. Libraries like next-intl handle both elegantly.

## Resources

- [ICU Message Format](https://unicode-org.github.io/icu/userguide/format_parse/messages/) — ICU Message Format specification

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*