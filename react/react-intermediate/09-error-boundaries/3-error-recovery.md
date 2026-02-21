---
source_course: "react-intermediate"
source_lesson: "react-error-recovery"
---

# Error Recovery Strategies

## Introduction

Catching errors is only half the battle. The other half is helping users recover gracefully. This lesson covers practical strategies for recovering from errors in React applications, from simple retries to sophisticated state reset patterns.

## Key Concepts

- **Error recovery**: The process of returning the application to a working state after an error occurs.
- **State reset**: Clearing the corrupted state that caused the error so the component can render successfully on retry.
- **Graceful degradation**: Showing a simpler version of the UI when the full version fails.

## Real World Context

Consider a social media feed. A post with malformed data causes the feed component to crash. With proper error recovery, the boundary catches the error, logs it, and shows a "Something went wrong" message with a "Refresh feed" button. Clicking the button resets the boundary, clears the local cache for that post, and re-fetches fresh data.

## Deep Dive

There are several recovery strategies depending on the type of error and the component architecture.

**Strategy 1: Simple Retry** — Reset the error boundary state and let React attempt to re-render the children. Works when errors are transient, such as a race condition or a temporary bad state.

```tsx
function FallbackWithRetry({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Failed to load: {error.message}</p>
      <button onClick={resetErrorBoundary}>Retry</button>
    </div>
  );
}
```

**Strategy 2: Reset with State Cleanup** — Before retrying, clear the state or cache that caused the error. This is useful when corrupted data triggered the crash.

```tsx
<ErrorBoundary
  FallbackComponent={FallbackWithRetry}
  onReset={() => {
    queryClient.invalidateQueries({ queryKey: ['feed'] });
  }}
>
  <Feed />
</ErrorBoundary>
```

**Strategy 3: Reset on Navigation** — Automatically reset the boundary when the user navigates to a different page. Use the `resetKeys` prop pattern to trigger a reset when certain values change.

```tsx
<ErrorBoundary
  FallbackComponent={PageError}
  resetKeys={[pathname]}
>
  <PageContent />
</ErrorBoundary>
```

**Strategy 4: Graceful Degradation** — Instead of retrying the same component, fall back to a simpler version that is less likely to fail.

## Common Pitfalls

1. **Infinite retry loops** — If the error is deterministic (bad data), retrying without clearing state causes the same crash repeatedly. Always clear corrupted state before retrying.
2. **Not resetting on navigation** — Users navigate away and back, but the error boundary still shows the fallback from the previous visit.

## Best Practices

1. **Combine retry with cache invalidation** — When using data-fetching libraries, invalidate the relevant queries before resetting the boundary.
2. **Use resetKeys for automatic recovery** — Pass route parameters or key props so the boundary resets automatically when the context changes.

## Summary

- Simple retries work for transient errors; combine with state cleanup for deterministic ones.
- Use `resetKeys` to automatically recover on navigation or prop changes.
- Graceful degradation provides a fallback experience when the full feature cannot recover.

## Code Examples

**Error recovery with automatic reset on navigation and cache invalidation**

```tsx
import { ErrorBoundary } from 'react-error-boundary';
import { useQueryClient } from '@tanstack/react-query';
import { usePathname } from 'next/navigation';

function PageErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <h2>This page encountered an error</h2>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function AppLayout({ children }) {
  const pathname = usePathname();
  const queryClient = useQueryClient();

  return (
    <ErrorBoundary
      FallbackComponent={PageErrorFallback}
      resetKeys={[pathname]}
      onReset={() => {
        queryClient.invalidateQueries();
      }}
    >
      {children}
    </ErrorBoundary>
  );
}
```


## Resources

- [Catching rendering errors with an error boundary](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) — Official React error boundary documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*