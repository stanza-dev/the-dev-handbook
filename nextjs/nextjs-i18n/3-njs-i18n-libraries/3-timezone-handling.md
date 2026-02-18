---
source_course: "nextjs-i18n"
source_lesson: "nextjs-i18n-njs-timezone-handling"
---

# Timezone Handling

## Introduction

Dates are tricky in i18n. A user in Tokyo and one in New York see the same timestamp differently. Proper timezone handling prevents confusion.

## Key Concepts

**Timezone considerations**:

- Store dates in UTC
- Display in user's local timezone
- Consider daylight saving time

## Deep Dive

### Storing Dates

```typescript
// Always store as UTC
const createdAt = new Date().toISOString();
// "2024-01-15T10:30:00.000Z"
```

### Displaying with User's Timezone

```typescript
import { useFormatter } from 'next-intl';

function EventTime({ date }: { date: string }) {
  const format = useFormatter();
  
  return (
    <time dateTime={date}>
      {format.dateTime(new Date(date), {
        dateStyle: 'long',
        timeStyle: 'short',
        timeZone: Intl.DateTimeFormat().resolvedOptions().timeZone,
      })}
    </time>
  );
}
// Tokyo: "January 15, 2024 at 7:30 PM"
// New York: "January 15, 2024 at 5:30 AM"
```

### Explicit Timezone Display

```typescript
format.dateTime(date, {
  timeZoneName: 'short',
});
// "Jan 15, 2024, 5:30 AM EST"
```

### Relative Time

```typescript
format.relativeTime(new Date(date));
// "2 hours ago"
// "in 3 days"
```

## Summary

Store all dates in UTC, display in the user's local timezone using Intl APIs. Include timezone indicators for ambiguous dates like meeting times.

## Resources

- [Intl.DateTimeFormat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl/DateTimeFormat) — JavaScript date formatting API

---

> 📘 *This lesson is part of the [Next.js Internationalization](https://stanza.dev/courses/nextjs-i18n) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*