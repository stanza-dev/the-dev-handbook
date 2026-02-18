---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-why-tanstack-query"
---

# Why TanStack Query?

TanStack Query (formerly React Query) solves the hard problems of server state management.

## The Problems It Solves

### 1. Caching

Without a library:
```jsx
// Every component fetches independently
function UserAvatar() {
  const { data } = useFetch('/api/user'); // Fetch #1
}

function UserProfile() {
  const { data } = useFetch('/api/user'); // Fetch #2 - Duplicate!
}
```

With TanStack Query:
```jsx
// Same key = shared cache
function UserAvatar() {
  const { data } = useQuery({ queryKey: ['user'], queryFn: fetchUser });
}

function UserProfile() {
  const { data } = useQuery({ queryKey: ['user'], queryFn: fetchUser });
  // Uses cached data! No duplicate request
}
```

### 2. Background Updates

Automatically refresh stale data:
- When window regains focus
- When network reconnects
- On interval
- On user action

### 3. Optimistic Updates

```jsx
const mutation = useMutation({
  mutationFn: updateTodo,
  onMutate: async (newTodo) => {
    // Cancel outgoing refetches
    await queryClient.cancelQueries(['todos']);
    
    // Snapshot previous value
    const previousTodos = queryClient.getQueryData(['todos']);
    
    // Optimistically update
    queryClient.setQueryData(['todos'], old => [...old, newTodo]);
    
    return { previousTodos };
  },
  onError: (err, newTodo, context) => {
    // Rollback on error
    queryClient.setQueryData(['todos'], context.previousTodos);
  },
});
```

### 4. Automatic Garbage Collection

Unused cache entries are cleaned up automatically.

### 5. DevTools

```jsx
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <MyApp />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

## Setup

```bash
npm install @tanstack/react-query
```

```jsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

const queryClient = new QueryClient();

function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <YourApp />
    </QueryClientProvider>
  );
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*