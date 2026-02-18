---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-error-handling-patterns"
---

# Error Handling Patterns

Robust error handling improves user experience significantly.

## Component-Level Errors

```jsx
function UserProfile({ userId }) {
  const { data, isError, error, refetch } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  if (isError) {
    return (
      <div className="error-container">
        <h3>Failed to load user</h3>
        <p>{error.message}</p>
        <button onClick={() => refetch()}>Try Again</button>
      </div>
    );
  }

  return <Profile user={data} />;
}
```

## Error Boundaries

Catch errors declaratively:

```jsx
import { ErrorBoundary } from 'react-error-boundary';
import { useQueryErrorResetBoundary } from '@tanstack/react-query';

function App() {
  const { reset } = useQueryErrorResetBoundary();

  return (
    <ErrorBoundary
      onReset={reset}
      fallbackRender={({ error, resetErrorBoundary }) => (
        <div className="error-page">
          <h1>Something went wrong</h1>
          <pre>{error.message}</pre>
          <button onClick={resetErrorBoundary}>Try again</button>
        </div>
      )}
    >
      <UserProfile />
    </ErrorBoundary>
  );
}
```

## Enable Throwing Errors

```jsx
// Make queries throw to error boundary
const { data } = useQuery({
  queryKey: ['user'],
  queryFn: fetchUser,
  throwOnError: true,  // Throw to nearest error boundary
});
```

## HTTP Error Handling

```jsx
const fetchUser = async (userId) => {
  const response = await fetch(`/api/users/${userId}`);
  
  if (!response.ok) {
    // Create error with helpful info
    const error = new Error('Failed to fetch user');
    error.status = response.status;
    error.statusText = response.statusText;
    
    // Try to get error message from response
    try {
      const body = await response.json();
      error.message = body.message || error.message;
    } catch {}
    
    throw error;
  }
  
  return response.json();
};

// Usage
if (error.status === 404) {
  return <NotFound />;
}
if (error.status === 403) {
  return <AccessDenied />;
}
```

## Global Error Handler

```jsx
const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (error, query) => {
      // Global error handling
      if (error.status === 401) {
        // Redirect to login
        window.location.href = '/login';
      }
      
      // Log to error service
      errorService.log(error, { queryKey: query.queryKey });
    },
  }),
});
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*