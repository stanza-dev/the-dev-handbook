---
source_course: "javascript"
source_lesson: "javascript-temporal-api"
---

# Modern Date Handling

## Introduction

The Temporal API is JavaScript's modern answer to Date's problems. It provides immutable, timezone-aware, type-safe date handling. While still being standardized, understanding Temporal prepares you for the future of JavaScript dates.

## Key Concepts

**Temporal.Instant**: A fixed point in time (like a timestamp).

**Temporal.PlainDate**: A date without time or timezone.

**Temporal.ZonedDateTime**: A date-time with timezone.

## Real World Context

Scheduling across timezones, handling DST transitions, calendar calculations—Temporal solves problems that plague Date users daily.

## Deep Dive

### Temporal Types Overview

```javascript
// Note: Temporal may require polyfill as of 2025

// Fixed instant in time
Temporal.Instant.from('2024-12-25T00:00:00Z');

// Plain (no timezone) types
Temporal.PlainDate.from('2024-12-25');
Temporal.PlainTime.from('10:30:00');
Temporal.PlainDateTime.from('2024-12-25T10:30:00');

// With timezone
Temporal.ZonedDateTime.from('2024-12-25T10:30:00[America/New_York]');

// Current time
Temporal.Now.instant();
Temporal.Now.zonedDateTimeISO();
Temporal.Now.plainDateISO();
```

### Immutable Operations

```javascript
const date = Temporal.PlainDate.from('2024-12-25');

// Returns new object (immutable!)
const nextMonth = date.add({ months: 1 });  // 2025-01-25
const lastWeek = date.subtract({ days: 7 }); // 2024-12-18

date.toString();  // Still '2024-12-25'

// Modifying specific fields
const newDate = date.with({ day: 1 });  // 2024-12-01
```

### Duration and Comparison

```javascript
const start = Temporal.PlainDate.from('2024-01-01');
const end = Temporal.PlainDate.from('2024-12-25');

// Get duration between dates
const duration = start.until(end);
duration.toString();  // 'P11M24D' (11 months, 24 days)
duration.days;        // Total days representation

// Comparison
Temporal.PlainDate.compare(start, end);  // -1 (start < end)
start.equals(end);  // false
```

### Timezone Handling

```javascript
const nyTime = Temporal.ZonedDateTime.from(
  '2024-12-25T10:00:00[America/New_York]'
);

// Convert to another timezone
const londonTime = nyTime.withTimeZone('Europe/London');
console.log(londonTime.toString());
// '2024-12-25T15:00:00+00:00[Europe/London]'

// Get offset
nyTime.offset;  // '-05:00'
```

### Until Temporal is Available

```javascript
// Use libraries that follow Temporal patterns:
// - Luxon
// - date-fns
// - js-joda

// Example with Luxon (similar API)
import { DateTime } from 'luxon';

DateTime.now().setZone('America/New_York');
DateTime.fromISO('2024-12-25').plus({ months: 1 });
```

## Common Pitfalls

1. **Temporal not yet standard**: Check browser support, use polyfills.
2. **Mixing Temporal and Date**: Keep consistent within codebase.
3. **Forgetting immutability**: Methods return new objects.

## Best Practices

- **Use PlainDate for birthdays**: Doesn't change with timezone.
- **Use ZonedDateTime for events**: Includes timezone context.
- **Use Instant for timestamps**: Absolute moment in time.
- **Use libraries until Temporal is standard**: Luxon mirrors Temporal's API.

## Summary

Temporal provides immutable, timezone-aware date handling. Different types serve different purposes: Instant for timestamps, PlainDate for dates, ZonedDateTime for events. Operations return new objects. Until standardized, use libraries like Luxon that follow similar patterns.

## Code Examples

**Temporal Types Overview**

```javascript
// Note: Temporal may require polyfill as of 2025

// Fixed instant in time
Temporal.Instant.from('2024-12-25T00:00:00Z');

// Plain (no timezone) types
Temporal.PlainDate.from('2024-12-25');
Temporal.PlainTime.from('10:30:00');
Temporal.PlainDateTime.from('2024-12-25T10:30:00');

// With timezone
Temporal.ZonedDateTime.from('2024-12-25T10:30:00[America/New_York]');

// Current time
Temporal.Now.instant();
Temporal.Now.zonedDateTimeISO();
Temporal.Now.plainDateISO();
```

**Immutable Operations**

```javascript
const date = Temporal.PlainDate.from('2024-12-25');

// Returns new object (immutable!)
const nextMonth = date.add({ months: 1 });  // 2025-01-25
const lastWeek = date.subtract({ days: 7 }); // 2024-12-18

date.toString();  // Still '2024-12-25'

// Modifying specific fields
const newDate = date.with({ day: 1 });  // 2024-12-01
```


## Resources

- [MDN: Temporal](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Temporal) — Temporal API reference
- [TC39 Temporal Proposal](https://tc39.es/proposal-temporal/docs/) — Official Temporal documentation

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*