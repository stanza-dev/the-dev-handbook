---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-error-handling"
---

# Error Handling Strategy

Consistent error handling across the app.

## Custom Error Classes

```tsx
// errors/index.ts
export class APIError extends Error {
  constructor(
    public status: number,
    message: string,
    public code?: string
  ) {
    super(message);
    this.name = 'APIError';
  }
  
  get isNotFound() {
    return this.status === 404;
  }
  
  get isUnauthorized() {
    return this.status === 401;
  }
  
  get isForbidden() {
    return this.status === 403;
  }
  
  get isValidationError() {
    return this.status === 422;
  }
}

export class NetworkError extends Error {
  constructor() {
    super('Network error. Please check your connection.');
    this.name = 'NetworkError';
  }
}
```

## Global Error Handler

```tsx
// api/client.ts
async function request<T>(endpoint: string, options: RequestInit): Promise<T> {
  try {
    const response = await fetch(endpoint, options);
    
    if (!response.ok) {
      const data = await response.json().catch(() => ({}));
      throw new APIError(response.status, data.message, data.code);
    }
    
    return response.json();
  } catch (error) {
    if (error instanceof APIError) {
      throw error;
    }
    if (error instanceof TypeError && error.message === 'Failed to fetch') {
      throw new NetworkError();
    }
    throw error;
  }
}
```

## React Query Error Handling

```tsx
// providers/QueryProvider.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { APIError } from '@/errors';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      retry: (failureCount, error) => {
        // Don't retry on 4xx errors
        if (error instanceof APIError && error.status >= 400 && error.status < 500) {
          return false;
        }
        return failureCount < 3;
      },
    },
    mutations: {
      onError: (error) => {
        if (error instanceof APIError && error.isUnauthorized) {
          // Redirect to login
          window.location.href = '/login';
        }
      },
    },
  },
});
```

## Error Boundary

```tsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <h2>Something went wrong</h2>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => window.location.reload()}
    >
      <AppContent />
    </ErrorBoundary>
  );
}
```

## Component-Level Error Handling

```tsx
function UserProfile({ userId }) {
  const { data: user, error, isError } = useUser(userId);
  
  if (isError) {
    if (error instanceof APIError) {
      if (error.isNotFound) {
        return <NotFound message="User not found" />;
      }
      if (error.isForbidden) {
        return <AccessDenied />;
      }
    }
    return <ErrorMessage error={error} />;
  }
  
  return <Profile user={user} />;
}
```

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*