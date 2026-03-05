---
source_course: "javascript"
source_lesson: "javascript-date-basics"
---

# The Date Object

## Introduction

Working with dates and times is notoriously tricky in programming. JavaScript's `Date` object, while imperfect, provides the tools you need for basic date manipulation. Understanding its quirks is essential for any web developer.

## Key Concepts

**Date Object**: Represents a single moment in time as milliseconds since January 1, 1970 UTC (Unix epoch).

**Timestamp**: The number of milliseconds since the epoch.

**UTC vs Local**: Dates can be interpreted in UTC or the user's local timezone.

## Real World Context

Displaying user-friendly dates, calculating durations, scheduling events, handling timezones—dates appear throughout applications. Getting them right is critical for user trust.

## Deep Dive

### Creating Dates

```javascript
// Current date/time
const now = new Date();

// From string (ISO 8601 format recommended)
const date1 = new Date('2024-12-25');  // Midnight UTC
const date2 = new Date('2024-12-25T10:30:00');  // Local time
const date3 = new Date('2024-12-25T10:30:00Z'); // UTC

// From components (months are 0-indexed!)
const date4 = new Date(2024, 11, 25);  // Dec 25, 2024
const date5 = new Date(2024, 11, 25, 10, 30, 0);  // With time

// From timestamp
const date6 = new Date(1703505600000);
```

### Getting Date Components

```javascript
const date = new Date('2024-12-25T10:30:45');

date.getFullYear();    // 2024
date.getMonth();       // 11 (0-indexed! December = 11)
date.getDate();        // 25 (day of month)
date.getDay();         // 3 (day of week, 0=Sunday)
date.getHours();       // 10
date.getMinutes();     // 30
date.getSeconds();     // 45
date.getMilliseconds(); // 0
date.getTime();        // Timestamp in ms

// UTC versions
date.getUTCFullYear();
date.getUTCMonth();
// ... etc
```

### Setting Date Components

```javascript
const date = new Date();

date.setFullYear(2025);
date.setMonth(5);       // June (0-indexed)
date.setDate(15);       // Day of month
date.setHours(14, 30, 0); // Can set multiple

// Overflow is handled automatically
date.setDate(32);  // Rolls over to next month
```

### Date Arithmetic

```javascript
const now = new Date();

// Add days
const tomorrow = new Date(now);
tomorrow.setDate(now.getDate() + 1);

// Difference in days
const date1 = new Date('2024-01-01');
const date2 = new Date('2024-12-31');
const diffMs = date2 - date1;
const diffDays = diffMs / (1000 * 60 * 60 * 24); // 365

// Comparing dates
date1 < date2;           // true
date1.getTime() === date2.getTime();  // Compare equality
```

### Static Methods

```javascript
Date.now();                    // Current timestamp
Date.parse('2024-12-25');      // Parse to timestamp
Date.UTC(2024, 11, 25, 10, 0); // Create UTC timestamp
```

## Common Pitfalls

1. **Months are 0-indexed**: January is 0, December is 11.
2. **String parsing inconsistency**: Different browsers parse differently. Use ISO 8601.
3. **Timezone confusion**: `new Date('2024-12-25')` is UTC; component constructor is local.

## Best Practices

- **Use ISO 8601 strings**: `YYYY-MM-DDTHH:mm:ssZ` for consistency.
- **Store dates in UTC**: Convert to local only for display.
- **Consider libraries**: date-fns or Luxon for complex operations.
- **Use getTime() for comparison**: Not direct comparison.

## Summary

Date objects represent moments in time. Months are 0-indexed (0-11). Use ISO 8601 format for reliable parsing. Get components with `getFullYear()`, `getMonth()`, etc. Arithmetic works through timestamps. For production apps, consider modern date libraries.

## Code Examples

**Creating Dates**

```javascript
// Current date/time
const now = new Date();

// From string (ISO 8601 format recommended)
const date1 = new Date('2024-12-25');  // Midnight UTC
const date2 = new Date('2024-12-25T10:30:00');  // Local time
const date3 = new Date('2024-12-25T10:30:00Z'); // UTC

// From components (months are 0-indexed!)
const date4 = new Date(2024, 11, 25);  // Dec 25, 2024
const date5 = new Date(2024, 11, 25, 10, 30, 0);  // With time

// From timestamp
const date6 = new Date(1703505600000);
```

**Getting Date Components**

```javascript
const date = new Date('2024-12-25T10:30:45');

date.getFullYear();    // 2024
date.getMonth();       // 11 (0-indexed! December = 11)
date.getDate();        // 25 (day of month)
date.getDay();         // 3 (day of week, 0=Sunday)
date.getHours();       // 10
date.getMinutes();     // 30
date.getSeconds();     // 45
date.getMilliseconds(); // 0
date.getTime();        // Timestamp in ms

// UTC versions
date.getUTCFullYear();
date.getUTCMonth();
// ... etc
```


## Resources

- [MDN: Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date) — Complete Date reference
- [MDN: Date.prototype.getMonth()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/getMonth) — getMonth() reference (note 0-indexing)

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*