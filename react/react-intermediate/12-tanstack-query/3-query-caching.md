---
source_course: "react-intermediate"
source_lesson: "react-query-caching"
---

# Query Caching and Stale Time

## Introduction

TanStack Query's caching system is what makes it dramatically better than manual fetching. Understanding how cache lifetimes, stale times, and garbage collection work allows you to fine-tune the balance between data freshness and network efficiency.

## Key Concepts

- **staleTime**: How long data is considered fresh. While fresh, useQuery returns cached data without refetching. Defaults to 0 (immediately stale).
- **gcTime** (formerly cacheTime): How long unused cache entries are kept in memory before garbage collection. Defaults to 5 minutes.
- **Refetch triggers**: TanStack Query can automatically refetch on window focus, network reconnection, and at regular intervals.

## Real World Context

A news site has articles that update rarely (staleTime: 10 minutes), stock prices that update constantly (staleTime: 0, refetchInterval: 5 seconds), and user profile data that almost never changes (staleTime: 30 minutes). Each query configures its own caching behavior based on how frequently the underlying data changes.

## Deep Dive

The relationship between staleTime and gcTime determines the caching behavior:

\`\`\`tsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // 5 minutes default
      gcTime: 10 * 60 * 1000,   // 10 minutes
    },
  },
});

// Per-query override
function StockPrice({ symbol }: { symbol: string }) {
  const { data } = useQuery({
    queryKey: ['stock', symbol],
    queryFn: () => fetchStockPrice(symbol),
    staleTime: 0,
    refetchInterval: 5000,
  });

  return <span>{data?.price}</span>;
}

function UserProfile() {
  const { data } = useQuery({
    queryKey: ['user', 'profile'],
    queryFn: fetchProfile,
    staleTime: 30 * 60 * 1000, // 30 minutes
  });

  return <h1>{data?.name}</h1>;
}
\`\`\`

Key behaviors to understand:

1. **Fresh data**: When staleTime has not elapsed, navigating to a component that uses this query shows cached data instantly with no network request.
2. **Stale data**: After staleTime elapses, the cached data is still shown immediately, but a background refetch is triggered. Once fresh data arrives, the UI updates seamlessly.
3. **No data**: If the query has never been fetched or was garbage collected, a full loading state is shown while fetching.

Background refetch triggers include window focus (\`refetchOnWindowFocus\`), mount (\`refetchOnMount\`), and reconnection (\`refetchOnReconnect\`). All are configurable.

## Common Pitfalls

1. **Leaving staleTime at 0 for rarely-changing data** — Every navigation triggers a refetch even when the data hasn't changed, wasting bandwidth.
2. **Setting gcTime too low** — Cached data is discarded too quickly, causing full loading states when users navigate back.

## Best Practices

1. **Set staleTime based on data change frequency** — Use 0 for real-time data, minutes for moderate data, and longer for rarely-changing data.
2. **Use select to transform data** — The \`select\` option transforms cached data without refetching, useful for deriving computed values from the same query.

## Summary

- staleTime controls how long data is considered fresh (no refetch needed).
- gcTime controls how long unused cache entries survive in memory.
- Configure per-query based on how frequently the underlying data changes.

## Code Examples

**Configuring staleTime for different data freshness needs**

```tsx
import { useQuery } from '@tanstack/react-query';

// Rarely-changing data: long stale time
function UserSettings() {
  const { data } = useQuery({
    queryKey: ['settings'],
    queryFn: fetchSettings,
    staleTime: 30 * 60 * 1000, // 30 min
  });
  return <SettingsForm defaults={data} />;
}

// Frequently-changing data: short stale time + polling
function LiveScores() {
  const { data } = useQuery({
    queryKey: ['scores'],
    queryFn: fetchScores,
    staleTime: 0,
    refetchInterval: 5000, // poll every 5s
  });
  return <Scoreboard scores={data} />;
}
```


## Resources

- [Caching](https://tanstack.com/query/latest/docs/framework/react/guides/caching) — Official TanStack Query caching guide

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*