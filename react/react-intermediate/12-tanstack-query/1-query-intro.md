---
source_course: "react-intermediate"
source_lesson: "react-tanstack-query-intro"
---

# Introduction to TanStack Query

## Introduction

TanStack Query (formerly React Query) is a powerful library for managing server state in React applications. It eliminates the boilerplate of manual data fetching, caching, synchronization, and background updates, letting you focus on building features instead of managing loading states.

## Key Concepts

- **Server state**: Data that lives on the server and is accessed asynchronously. Unlike client state (UI state, form inputs), server state is shared, can become stale, and requires synchronization.
- **Query**: A declarative dependency on an asynchronous data source, identified by a unique key.
- **Query key**: A serializable array that uniquely identifies a query for caching and refetching.
- **Stale-while-revalidate**: A caching strategy where stale data is shown immediately while fresh data is fetched in the background.

## Real World Context

A project management app needs to show a list of tasks that multiple users can edit simultaneously. With manual fetch and useState, you would need to handle loading, error, caching, refetching on focus, polling for updates, and invalidation after mutations. TanStack Query handles all of these with a single useQuery call.

## Deep Dive

TanStack Query requires a QueryClient and QueryClientProvider at the root of your app. The \`useQuery\` hook accepts a query key and a fetch function, returning the data along with loading, error, and other metadata.

\`\`\`tsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
} from '@tanstack/react-query';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoList />
    </QueryClientProvider>
  );
}

function TodoList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['todos'],
    queryFn: async () => {
      const res = await fetch('/api/todos');
      if (!res.ok) throw new Error('Failed to fetch');
      return res.json();
    },
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
}
\`\`\`

Query keys can be hierarchical arrays, enabling targeted invalidation. For example, \`['todos', { status: 'done' }]\` is a separate cache entry from \`['todos']\`, but invalidating \`['todos']\` invalidates both.

TanStack Query provides automatic background refetching on window focus, configurable stale times, and garbage collection of unused queries.

## Common Pitfalls

1. **Putting fetch logic in useEffect with TanStack Query** — The whole point is that useQuery replaces the useEffect + useState pattern. Do not wrap useQuery inside useEffect.
2. **Using unstable query keys** — If the query key includes a new object reference every render, TanStack Query treats it as a new query and refetches unnecessarily.

## Best Practices

1. **Use structured query keys** — Organize keys hierarchically: \`['entity', id, 'subresource']\` for easy targeted invalidation.
2. **Configure staleTime based on data freshness needs** — Set it to 0 for real-time data, or longer (e.g., 5 minutes) for data that changes infrequently.

## Summary

- TanStack Query manages server state with automatic caching, refetching, and synchronization.
- useQuery replaces the manual useEffect + useState pattern for data fetching.
- Query keys enable hierarchical caching and targeted invalidation.

## Code Examples

**Basic TanStack Query setup with useQuery**

```tsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
} from '@tanstack/react-query';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoList />
    </QueryClientProvider>
  );
}

function TodoList() {
  const { data, isLoading, error } = useQuery({
    queryKey: ['todos'],
    queryFn: () => fetch('/api/todos').then(r => r.json()),
  });

  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map(todo => <li key={todo.id}>{todo.title}</li>)}
    </ul>
  );
}
```


## Resources

- [TanStack Query Documentation](https://tanstack.com/query/latest) — Official TanStack Query docs

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*