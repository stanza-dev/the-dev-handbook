---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-retry-strategies"
---

# Retry Strategies

Automatic retries can recover from transient failures.

## Default Retry Behavior

TanStack Query retries failed queries 3 times by default:

```jsx
// Default behavior
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  retry: 3,  // Default
});
```

## Custom Retry Count

```jsx
useQuery({
  queryKey: ['critical-data'],
  queryFn: fetchCriticalData,
  retry: 5,  // More retries for important data
});

useQuery({
  queryKey: ['user-action'],
  queryFn: submitForm,
  retry: 0,  // No retries for user actions
});
```

## Conditional Retry

```jsx
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  retry: (failureCount, error) => {
    // Don't retry on 4xx errors (client errors)
    if (error.status >= 400 && error.status < 500) {
      return false;
    }
    // Retry up to 3 times for other errors
    return failureCount < 3;
  },
});
```

## Retry Delay

```jsx
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  retryDelay: (attemptIndex) => {
    // Exponential backoff: 1s, 2s, 4s, 8s...
    return Math.min(1000 * 2 ** attemptIndex, 30000);
  },
});

// Or fixed delay
useQuery({
  queryKey: ['data'],
  queryFn: fetchData,
  retryDelay: 3000,  // Always wait 3 seconds
});
```

## Retry for Mutations

```jsx
useMutation({
  mutationFn: createTodo,
  retry: 3,
  retryDelay: 1000,
});
```

## Network Status Aware

```jsx
function useNetworkAwareQuery(queryKey, queryFn) {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return useQuery({
    queryKey,
    queryFn,
    enabled: isOnline,  // Don't try when offline
    retry: isOnline ? 3 : 0,
  });
}
```

## Global Retry Configuration

```jsx
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: 3,
      retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000),
    },
    mutations: {
      retry: 0,  // Don't retry mutations by default
    },
  },
});
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*