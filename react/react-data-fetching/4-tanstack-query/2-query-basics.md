---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-use-query-basics"
---

# useQuery Basics

`useQuery` is the primary hook for fetching data.

## Basic Usage

```jsx
import { useQuery } from '@tanstack/react-query';

function UserProfile({ userId }) {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  if (isLoading) return <Spinner />;
  if (isError) return <Error message={error.message} />;
  
  return <Profile user={data} />;
}
```

## Query Keys

Query keys uniquely identify cached data:

```jsx
// Simple key
useQuery({ queryKey: ['todos'], queryFn: fetchTodos });

// Key with parameters
useQuery({ queryKey: ['todo', todoId], queryFn: () => fetchTodo(todoId) });

// Key with filters
useQuery({ 
  queryKey: ['todos', { status: 'done', page: 1 }],
  queryFn: () => fetchTodos({ status: 'done', page: 1 })
});
```

## Query Function

Must return a promise:

```jsx
const { data } = useQuery({
  queryKey: ['user', userId],
  queryFn: async () => {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    return response.json();
  },
});
```

## Return Values

```jsx
const {
  data,        // The resolved data
  error,       // Error object if failed
  isLoading,   // True on first fetch
  isFetching,  // True whenever fetching (including background)
  isError,     // True if query failed
  isSuccess,   // True if query succeeded
  refetch,     // Function to manually refetch
  status,      // 'pending' | 'error' | 'success'
} = useQuery({ queryKey, queryFn });
```

## Query Options

```jsx
const { data } = useQuery({
  queryKey: ['todos'],
  queryFn: fetchTodos,
  
  // Caching
  staleTime: 5 * 60 * 1000,     // Data fresh for 5 minutes
  gcTime: 30 * 60 * 1000,       // Keep unused data for 30 minutes
  
  // Refetching
  refetchOnWindowFocus: true,   // Refetch when tab gains focus
  refetchOnMount: true,         // Refetch on component mount
  refetchInterval: 60000,       // Poll every minute
  
  // Behavior
  enabled: !!userId,            // Only run if userId exists
  retry: 3,                     // Retry failed queries 3 times
  
  // Placeholders
  placeholderData: previousData => previousData,
  initialData: [],
});
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*