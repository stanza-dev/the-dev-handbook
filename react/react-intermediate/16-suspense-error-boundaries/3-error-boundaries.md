---
source_course: "react-intermediate"
source_lesson: "react-error-boundaries"
---

# Error Boundaries

## Introduction

Error boundaries are React components that catch JavaScript errors in their child component tree, log them, and display a fallback UI instead of crashing the whole application.

## Key Concepts

**Error boundary** is a class component that implements `static getDerivedStateFromError()` and/or `componentDidCatch()`. It catches errors during rendering, in lifecycle methods, and in constructors of child components.

## Real World Context

In production, error boundaries:
- Prevent a single component crash from breaking the entire app
- Display user-friendly error messages
- Enable error reporting to services like Sentry
- Allow users to retry or navigate away

## Deep Dive

### Creating an Error Boundary

Error boundaries must be class components:

```tsx
class ErrorBoundary extends Component<Props, State> {
  state = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error) {
    // Update state to show fallback UI
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log to error reporting service
    logErrorToService(error, errorInfo.componentStack);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}
```

### What Errors Are Caught?

**Caught:**
- Errors during rendering
- Errors in lifecycle methods
- Errors in constructors

**NOT Caught:**
- Event handlers (use try/catch)
- Asynchronous code (promises, setTimeout)
- Server-side rendering errors
- Errors in the error boundary itself

### Strategic Placement

Place boundaries at different granularities:

```tsx
// App-level - catches everything, shows full-page error
<ErrorBoundary fallback={<FullPageError />}>
  <App />
</ErrorBoundary>

// Feature-level - isolates failures to features
<ErrorBoundary fallback={<WidgetError />}>
  <Dashboard />
</ErrorBoundary>

// Component-level - for risky third-party widgets
<ErrorBoundary fallback={<ChartError />}>
  <ThirdPartyChart />
</ErrorBoundary>
```

### Recovery Patterns

Allow users to retry:

```tsx
class ErrorBoundary extends Component {
  state = { hasError: false };

  resetError = () => {
    this.setState({ hasError: false });
  };

  render() {
    if (this.state.hasError) {
      return (
        <ErrorFallback
          onRetry={this.resetError}
        />
      );
    }
    return this.props.children;
  }
}
```

## Common Pitfalls

1. **Using function components**: Error boundaries MUST be class components - there's no hook equivalent (yet).
2. **Catching event handler errors**: Error boundaries don't catch these - use try/catch in event handlers.
3. **Single boundary at root only**: This provides no isolation - a single error crashes everything.

## Best Practices

- Place multiple boundaries at strategic points in your component tree
- Match fallback UI complexity to what failed (widget error vs page error)
- Log errors to a monitoring service (Sentry, LogRocket, etc.)
- Provide recovery options when possible (retry button, navigation)
- Consider using react-error-boundary library for a more flexible API
- Test error boundaries by intentionally throwing errors

## Summary

Error boundaries catch rendering errors and prevent them from crashing your entire app. Use class components with `getDerivedStateFromError` and `componentDidCatch`. Place boundaries strategically to isolate failures.

## Code Examples

**Error boundary with nested boundaries and error reporting**

```tsx
import { Component, ReactNode } from 'react';

type Props = {
  children: ReactNode;
  fallback: ReactNode;
  onError?: (error: Error, errorInfo: React.ErrorInfo) => void;
};

type State = {
  hasError: boolean;
  error: Error | null;
};

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log to error reporting service
    console.error('Error caught by boundary:', error);
    console.error('Component stack:', errorInfo.componentStack);
    
    // Call optional error handler
    this.props.onError?.(error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback;
    }
    return this.props.children;
  }
}

// Usage with different fallbacks
function App() {
  return (
    <ErrorBoundary
      fallback={<FullPageError />}
      onError={(error) => sendToSentry(error)}
    >
      <Header />
      
      <ErrorBoundary fallback={<DashboardError />}>
        <Dashboard />
      </ErrorBoundary>
      
      <ErrorBoundary fallback={<SidebarError />}>
        <Sidebar />
      </ErrorBoundary>
    </ErrorBoundary>
  );
}

function DashboardError() {
  return (
    <div className="error-card">
      <h2>Dashboard failed to load</h2>
      <p>Please try refreshing the page.</p>
      <button onClick={() => window.location.reload()}>
        Refresh
      </button>
    </div>
  );
}
```


## Resources

- [Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) — Official React error boundary documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*