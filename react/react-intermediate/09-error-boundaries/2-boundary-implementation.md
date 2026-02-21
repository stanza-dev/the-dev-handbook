---
source_course: "react-intermediate"
source_lesson: "react-error-boundary-implementation"
---

# Building a Reusable Error Boundary

## Introduction

While the basic error boundary pattern works, production applications need a more flexible, reusable implementation that supports custom fallbacks, error reporting, and recovery mechanisms. This lesson shows how to build one from scratch and when to use the popular `react-error-boundary` library instead.

## Key Concepts

- **getDerivedStateFromError**: A static lifecycle method called during the render phase. It receives the thrown error and returns an object to update state so the next render shows a fallback.
- **componentDidCatch**: Called during the commit phase after an error. Ideal for side effects like logging to an error reporting service.
- **Error recovery**: The ability to reset the boundary state and retry rendering the children.

## Real World Context

In a production e-commerce application, you might wrap the product detail page, the shopping cart, and the checkout form each in their own error boundary. If the product recommendation widget crashes due to bad API data, the rest of the page continues working. The error boundary logs to Sentry, shows a polite message, and offers a retry button.

## Deep Dive

A reusable error boundary typically accepts a `fallback` prop that can be a React node or a render function receiving the error and a reset callback. This pattern gives consumers full control over what to display when things go wrong.

```tsx
import { Component, ReactNode, ErrorInfo } from 'react';

type FallbackRender = (props: {
  error: Error;
  resetError: () => void;
}) => ReactNode;

type Props = {
  children: ReactNode;
  fallback?: ReactNode;
  fallbackRender?: FallbackRender;
  onError?: (error: Error, info: ErrorInfo) => void;
  onReset?: () => void;
};

type State = { hasError: boolean; error: Error | null };

class ReusableErrorBoundary extends Component<Props, State> {
  state: State = { hasError: false, error: null };

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    this.props.onError?.(error, info);
  }

  resetError = () => {
    this.props.onReset?.();
    this.setState({ hasError: false, error: null });
  };

  render() {
    if (this.state.hasError && this.state.error) {
      if (this.props.fallbackRender) {
        return this.props.fallbackRender({
          error: this.state.error,
          resetError: this.resetError,
        });
      }
      return this.props.fallback || <h2>Something went wrong</h2>;
    }
    return this.props.children;
  }
}
```

The `react-error-boundary` library provides this pattern out of the box with additional features like automatic reset on prop changes and `useErrorBoundary` hook for programmatic error throwing.

## Common Pitfalls

1. **Not providing a reset mechanism** — Users get stuck on the fallback with no way to recover. Always offer a retry or navigation option.
2. **Forgetting to log errors** — Without `componentDidCatch` side effects, errors silently disappear in production.

## Best Practices

1. **Use fallbackRender for dynamic fallbacks** — It gives you access to the error object and a reset function, enabling contextual error messages.
2. **Integrate with error monitoring** — Connect `onError` to services like Sentry or LogRocket for production visibility.

## Summary

- A reusable error boundary accepts flexible fallback props and supports error recovery.
- Use `componentDidCatch` for logging and `getDerivedStateFromError` for UI updates.
- The `react-error-boundary` library provides a production-ready implementation.

## Code Examples

**Using react-error-boundary library for flexible error handling**

```tsx
import { ErrorBoundary } from 'react-error-boundary';

function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div role="alert">
      <p>Something went wrong:</p>
      <pre>{error.message}</pre>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

function App() {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onError={(error, info) => logToSentry(error, info)}
      onReset={() => { /* clear app state */ }}
    >
      <Dashboard />
    </ErrorBoundary>
  );
}
```


## Resources

- [react-error-boundary](https://github.com/bvaughn/react-error-boundary) — Popular error boundary library with hooks and reset support

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*