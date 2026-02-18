---
source_course: "react"
source_lesson: "react-error-boundaries-intro"
---

# Error Boundaries

Error boundaries are React components that catch JavaScript errors anywhere in their child component tree and display a fallback UI.

## What They Catch

- Errors during rendering
- Errors in lifecycle methods
- Errors in constructors of the tree below them

## What They Don't Catch

- Event handlers (use try/catch)
- Async code (promises)
- Server-side rendering
- Errors in the error boundary itself

## Code Examples

**Custom Error Boundary class component**

```tsx
import { Component, ReactNode } from 'react';

type Props = {
  children: ReactNode;
  fallback?: ReactNode;
};

type State = {
  hasError: boolean;
  error?: Error;
};

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log to error reporting service
    console.error('Error caught:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            Try again
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary fallback={<div>Error loading widget</div>}>
      <MyWidget />
    </ErrorBoundary>
  );
}
```

**Using react-error-boundary library**

```tsx
// Using react-error-boundary library (recommended)
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
      onReset={() => {
        // Reset app state here
      }}
    >
      <MyComponent />
    </ErrorBoundary>
  );
}
```


## Resources

- [react-error-boundary](https://github.com/bvaughn/react-error-boundary) — Popular error boundary library

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*