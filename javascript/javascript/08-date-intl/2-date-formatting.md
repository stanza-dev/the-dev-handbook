---
source_course: "javascript"
source_lesson: "javascript-date-formatting"
---

# Date Formatting

## Introduction

Raw dates are ugly. Users expect friendly formats like "December 25, 2024" or "25/12/2024". JavaScript's `Intl.DateTimeFormat` provides powerful, locale-aware formatting without external libraries.

## Key Concepts

**Locale**: A string identifying language and region (e.g., 'en-US', 'fr-FR').

**Intl.DateTimeFormat**: API for locale-sensitive date formatting.

**Format Options**: Control which parts appear and how they're displayed.

## Real World Context

International applications must display dates appropriately for each user's locale. US users expect MM/DD/YYYY; Europeans expect DD/MM/YYYY. `Intl` handles this automatically.

## Deep Dive

### Basic Formatting

```javascript
const date = new Date('2024-12-25T10:30:00');

// toLocaleString variants
date.toLocaleDateString('en-US');  // '12/25/2024'
date.toLocaleDateString('en-GB');  // '25/12/2024'
date.toLocaleDateString('de-DE');  // '25.12.2024'
date.toLocaleTimeString('en-US');  // '10:30:00 AM'
date.toLocaleString('en-US');      // '12/25/2024, 10:30:00 AM'
```

### Intl.DateTimeFormat

```javascript
const date = new Date('2024-12-25T10:30:00');

// Create reusable formatter
const formatter = new Intl.DateTimeFormat('en-US', {
  weekday: 'long',
  year: 'numeric',
  month: 'long',
  day: 'numeric'
});

formatter.format(date);  // 'Wednesday, December 25, 2024'

// Format options
new Intl.DateTimeFormat('en-US', {
  dateStyle: 'full',   // 'Wednesday, December 25, 2024'
  timeStyle: 'short'   // '10:30 AM'
}).format(date);

// Individual parts
new Intl.DateTimeFormat('en-US', {
  year: 'numeric',     // '2024' or '24'
  month: 'long',       // 'December', 'short'='Dec', 'narrow'='D'
  day: '2-digit',      // '25'
  hour: 'numeric',     // '10' or '10 AM'
  minute: '2-digit',   // '30'
  hour12: true         // Use 12-hour format
}).format(date);
```

### Relative Time Formatting

```javascript
const rtf = new Intl.RelativeTimeFormat('en', { numeric: 'auto' });

rtf.format(-1, 'day');     // 'yesterday'
rtf.format(1, 'day');      // 'tomorrow'
rtf.format(-3, 'day');     // '3 days ago'
rtf.format(2, 'week');     // 'in 2 weeks'
rtf.format(-1, 'month');   // 'last month'

// Calculate and format
function timeAgo(date) {
  const seconds = Math.floor((Date.now() - date) / 1000);
  const intervals = [
    { unit: 'year', seconds: 31536000 },
    { unit: 'month', seconds: 2592000 },
    { unit: 'week', seconds: 604800 },
    { unit: 'day', seconds: 86400 },
    { unit: 'hour', seconds: 3600 },
    { unit: 'minute', seconds: 60 }
  ];
  
  for (const { unit, seconds: s } of intervals) {
    const count = Math.floor(seconds / s);
    if (count >= 1) return rtf.format(-count, unit);
  }
  return 'just now';
}
```

### Duration Formatting (ES2025/Temporal)

```javascript
// Using modern APIs (check browser support)
const duration = { hours: 2, minutes: 30 };

new Intl.DurationFormat('en', { style: 'long' })
  .format(duration);  // '2 hours, 30 minutes'
```

## Common Pitfalls

1. **Not specifying locale**: Results vary by user's system settings.
2. **Hardcoding formats**: Use Intl instead of string concatenation.
3. **Forgetting timezone**: Formatted dates use local timezone by default.

## Best Practices

- **Use Intl.DateTimeFormat**: Not manual formatting.
- **Create reusable formatters**: They're cached and efficient.
- **Use RelativeTimeFormat for "time ago"**: More natural than absolute dates.
- **Test with different locales**: Ensure layout handles different lengths.

## Summary

`Intl.DateTimeFormat` provides locale-aware date formatting. Use `dateStyle`/`timeStyle` for common formats or individual options for control. `RelativeTimeFormat` creates "3 days ago" style strings. Always prefer Intl over manual string building.

## Resources

- [MDN: Intl.DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat) — DateTimeFormat reference
- [MDN: Intl.RelativeTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/RelativeTimeFormat) — RelativeTimeFormat reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*