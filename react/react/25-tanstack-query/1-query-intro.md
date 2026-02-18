---
source_course: "react"
source_lesson: "react-tanstack-query-intro"
---

# TanStack Query (React Query)

A powerful library for fetching, caching, and managing server state.

## Benefits

- **Automatic caching**: No manual cache management
- **Background updates**: Stale-while-revalidate
- **Deduplication**: Multiple components, one request
- **DevTools**: Debug your queries easily

## Key Concepts

- **Queries**: For fetching data (GET)
- **Mutations**: For changing data (POST/PUT/DELETE)
- **Query Keys**: Unique identifiers for caching

## Code Examples

**TanStack Query basics**

```tsx
import {
  QueryClient,
  QueryClientProvider,
  useQuery,
  useMutation,
  useQueryClient
} from '@tanstack/react-query';

const queryClient = new QueryClient();

// Wrap app with provider
function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <TodoList />
    </QueryClientProvider>
  );
}

// Fetching data with useQuery
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

// Mutations for creating/updating data
function AddTodo() {
  const queryClient = useQueryClient();
  
  const mutation = useMutation({
    mutationFn: (newTodo: { title: string }) =>
      fetch('/api/todos', {
        method: 'POST',
        body: JSON.stringify(newTodo),
      }).then(res => res.json()),
    onSuccess: () => {
      // Invalidate and refetch
      queryClient.invalidateQueries({ queryKey: ['todos'] });
    },
  });

  return (
    <button
      onClick={() => mutation.mutate({ title: 'New Todo' })}
      disabled={mutation.isPending}
    >
      {mutation.isPending ? 'Adding...' : 'Add Todo'}
    </button>
  );
}
```


## Resources

- [TanStack Query Documentation](https://tanstack.com/query/latest) — Official TanStack Query docs

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*