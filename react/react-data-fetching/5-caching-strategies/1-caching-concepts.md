---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-caching-concepts"
---

# Caching Concepts

Understanding caching is crucial for performant data fetching.

## Why Cache?

1. **Performance**: Instant display of cached data
2. **Reduce Server Load**: Fewer API calls
3. **Offline Support**: Show data without network
4. **Better UX**: No loading spinners for repeat visits

## Cache Freshness

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   FRESH     │ -> │   STALE     │ -> │   GARBAGE   │
│ (staleTime) │    │ (gcTime)    │    │ (deleted)   │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Fresh Data
- Recently fetched
- Won't trigger refetch
- Displayed immediately

### Stale Data
- Past staleTime
- Displayed immediately BUT...
- Triggers background refetch

### Garbage Collection
- No active subscribers
- Past gcTime
- Removed from cache

## Stale-While-Revalidate

The SWR pattern:

1. Return cached (stale) data immediately
2. Revalidate in background
3. Update UI when fresh data arrives

```jsx
// User sees data instantly, gets fresh data shortly after
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 0,  // Consider stale immediately
});
```

## Cache Time vs Stale Time

```jsx
useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  staleTime: 5 * 60 * 1000,  // Fresh for 5 minutes
  gcTime: 30 * 60 * 1000,    // Keep in cache for 30 minutes
});
```

**Scenario**:
- User fetches todos at 10:00
- User navigates away at 10:02
- User returns at 10:04 (within staleTime)
  - Data displayed, NO refetch
- User returns at 10:10 (past staleTime, within gcTime)
  - Data displayed, background refetch
- User returns at 10:45 (past gcTime)
  - Loading spinner, fresh fetch

## Prefetching

```jsx
const queryClient = useQueryClient();

// Prefetch on hover
function TodoItem({ todo }) {
  const prefetchDetails = () => {
    queryClient.prefetchQuery({
      queryKey: ['todo', todo.id],
      queryFn: () => fetchTodoDetails(todo.id),
    });
  };

  return (
    <Link
      to={`/todo/${todo.id}`}
      onMouseEnter={prefetchDetails}
    >
      {todo.title}
    </Link>
  );
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*