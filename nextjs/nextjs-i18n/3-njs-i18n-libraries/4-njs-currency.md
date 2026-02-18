---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-currency"
---

# Currency Formatting

## Introduction

Currency display varies by locale: $1,234.56 in the US becomes 1.234,56 € in Germany. Getting this wrong can confuse customers and hurt sales.

## Key Concepts

**Currency formatting factors**:

- Symbol position (before/after)
- Decimal separator (. or ,)
- Thousands separator (. or , or space)
- Currency code vs symbol

## Deep Dive

### Basic Currency Formatting

```typescript
import { useFormatter } from 'next-intl';

function Price({ amount, currency }: { amount: number; currency: string }) {
  const format = useFormatter();
  
  return (
    <span>
      {format.number(amount, {
        style: 'currency',
        currency,
      })}
    </span>
  );
}

// en-US: $1,234.56
// de-DE: 1.234,56 $
// ja-JP: $1,234.56
```

### Currency Code vs Symbol

```typescript
format.number(amount, {
  style: 'currency',
  currency: 'EUR',
  currencyDisplay: 'code',    // EUR 1,234.56
  // or: 'symbol'             // €1,234.56
  // or: 'narrowSymbol'       // €1,234.56 (compact)
  // or: 'name'               // 1,234.56 euros
});
```

### Handling Multiple Currencies

```typescript
function MultiCurrencyPrice({ prices }: { prices: Record<string, number> }) {
  const format = useFormatter();
  const locale = useLocale();
  
  // Get user's preferred currency based on locale
  const localeCurrency = {
    'en-US': 'USD',
    'en-GB': 'GBP',
    'de-DE': 'EUR',
    'ja-JP': 'JPY',
  }[locale] || 'USD';
  
  return (
    <span>
      {format.number(prices[localeCurrency], {
        style: 'currency',
        currency: localeCurrency,
      })}
    </span>
  );
}
```

## Summary

Use Intl.NumberFormat with style: 'currency' for locale-aware currency display. Consider currency code display for international audiences and match currency to user's locale when possible.

## Resources

- [Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat) — JavaScript number formatting API

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*