---
source_course: "javascript"
source_lesson: "javascript-intl-numbers"
---

# Number & Currency Formatting

## Introduction

Numbers look different around the world: 1,234.56 in the US becomes 1.234,56 in Germany. Currency symbols, percentage signs, and unit formatting all vary by locale. `Intl.NumberFormat` handles this complexity.

## Key Concepts

**NumberFormat**: Locale-sensitive number formatting.

**Style**: 'decimal', 'currency', 'percent', or 'unit'.

**Notation**: 'standard', 'scientific', 'engineering', or 'compact'.

## Real World Context

E-commerce sites display prices in local currency formats. Analytics dashboards show large numbers compactly. Internationalized apps adapt number formatting to user preferences.

## Deep Dive

### Basic Number Formatting

```javascript
const num = 1234567.89;

new Intl.NumberFormat('en-US').format(num);  // '1,234,567.89'
new Intl.NumberFormat('de-DE').format(num);  // '1.234.567,89'
new Intl.NumberFormat('fr-FR').format(num);  // '1 234 567,89'
```

### Currency Formatting

```javascript
const price = 1234.56;

new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD'
}).format(price);  // '$1,234.56'

new Intl.NumberFormat('de-DE', {
  style: 'currency',
  currency: 'EUR'
}).format(price);  // '1.234,56 €'

new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY'
}).format(price);  // '￥1,235' (no decimals for yen)

// Currency display options
new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
  currencyDisplay: 'code'    // 'USD 1,234.56'
  // currencyDisplay: 'name' // '1,234.56 US dollars'
}).format(price);
```

### Percentage and Units

```javascript
// Percentage
new Intl.NumberFormat('en-US', {
  style: 'percent'
}).format(0.25);  // '25%'

// Units
new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'kilometer',
  unitDisplay: 'short'  // 'km', 'long', 'narrow'
}).format(100);  // '100 km'

new Intl.NumberFormat('en-US', {
  style: 'unit',
  unit: 'kilogram-per-meter'
}).format(5);  // '5 kg/m'
```

### Compact Notation

```javascript
new Intl.NumberFormat('en-US', {
  notation: 'compact'
}).format(1234567);  // '1.2M'

new Intl.NumberFormat('en-US', {
  notation: 'compact',
  compactDisplay: 'long'
}).format(1234567);  // '1.2 million'
```

### Decimal Precision

```javascript
const num = 1234.5678;

new Intl.NumberFormat('en-US', {
  minimumFractionDigits: 2,
  maximumFractionDigits: 2
}).format(num);  // '1,234.57'

new Intl.NumberFormat('en-US', {
  minimumSignificantDigits: 3,
  maximumSignificantDigits: 5
}).format(num);  // '1,234.6'
```

## Common Pitfalls

1. **Using toFixed for display**: Doesn't localize separators.
2. **Hardcoding currency symbols**: Different positions in different locales.
3. **Assuming decimal precision**: JPY has 0 decimals, BHD has 3.

## Best Practices

- **Always use Intl for user-facing numbers**: Never hardcode formats.
- **Cache formatters**: Create once, reuse many times.
- **Match currency to locale when appropriate**: Or use a separate currency selector.

## Summary

`Intl.NumberFormat` handles locale-aware number, currency, percent, and unit formatting. Use compact notation for large numbers. Specify precision with fraction/significant digit options. Always prefer Intl over manual formatting for user-facing numbers.

## Code Examples

**Basic Number Formatting**

```javascript
const num = 1234567.89;

new Intl.NumberFormat('en-US').format(num);  // '1,234,567.89'
new Intl.NumberFormat('de-DE').format(num);  // '1.234.567,89'
new Intl.NumberFormat('fr-FR').format(num);  // '1 234 567,89'
```

**Currency Formatting**

```javascript
const price = 1234.56;

new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD'
}).format(price);  // '$1,234.56'

new Intl.NumberFormat('de-DE', {
  style: 'currency',
  currency: 'EUR'
}).format(price);  // '1.234,56 €'

new Intl.NumberFormat('ja-JP', {
  style: 'currency',
  currency: 'JPY'
}).format(price);  // '￥1,235' (no decimals for yen)

// Currency display options
```


## Resources

- [MDN: Intl.NumberFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/NumberFormat) — NumberFormat reference
- [MDN: Intl](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl) — Intl object overview

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*