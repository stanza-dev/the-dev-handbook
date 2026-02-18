---
source_course: "nextjs-optimization"
source_lesson: "nextjs-optimization-njs-data-cache"
---

# Understanding the Data Cache

## Introduction

Next.js has multiple cache layers. Understanding how the Data Cache works helps you optimize fetch requests and avoid unexpected stale data.

## Key Concepts

**Cache layers**:

1. **Request Memoization**: Same fetch in one render is deduped
2. **Data Cache**: Persists across requests and deploys
3. **Full Route Cache**: Cached HTML/RSC payload
4. **Router Cache**: Client-side navigation cache

## Deep Dive

### Data Cache Behavior

```typescript
// Cached indefinitely by default
const data = await fetch('https://api.example.com/data');

// Revalidate after 60 seconds
const data = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 },
});

// Never cache (always fresh)
const data = await fetch('https://api.example.com/data', {
  cache: 'no-store',
});
```

### When to Opt Out

```typescript
// User-specific data
const profile = await fetch(`/api/user/${userId}`, {
  cache: 'no-store',
});

// Real-time data
const stockPrice = await fetch('/api/stocks/AAPL', {
  cache: 'no-store',
});
```

## Summary

The Data Cache persists fetch results across requests. Use `next: { revalidate }` for time-based freshness, `cache: 'no-store'` for dynamic data, and tags for on-demand invalidation.

## Resources

- [Data Cache](https://nextjs.org/docs/app/building-your-application/caching#data-cache) — Data Cache documentation

---

> 📘 *This lesson is part of the [Next.js Performance & Optimization](https://stanza.dev/courses/nextjs-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*