---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-error-boundaries"
---

# Error Boundaries: Catching Render Errors

Error Boundaries are React components that catch JavaScript errors in their child component tree, log those errors, and display a fallback UI.

## The Problem

Without error boundaries, a rendering error crashes the entire app:

```tsx
function BuggyComponent() {
  throw new Error('Oops!');
  return <div>Never renders</div>;
}

function App() {
  return (
    <div>
      <Header />
      <BuggyComponent /> {/* Crashes everything! */}
      <Footer />
    </div>
  );
}
```

## Creating an Error Boundary

Error boundaries must be class components (no hook equivalent yet):

```tsx
import { Component, ErrorInfo, ReactNode } from 'react';

type Props = {
  children: ReactNode;
  fallback?: ReactNode;
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
    // Update state to show fallback UI
    return { hasError: true, error };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    // Log error to reporting service
    console.error('Error caught:', error);
    console.error('Component stack:', errorInfo.componentStack);
    
    // Send to error tracking service
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? (
        <div className="error-fallback">
          <h2>Something went wrong</h2>
          <button onClick={() => this.setState({ hasError: false, error: null })}>
            Try again
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}
```

## Usage

```tsx
function App() {
  return (
    <ErrorBoundary fallback={<ErrorPage />}>
      <Header />
      <main>
        <ErrorBoundary fallback={<WidgetError />}>
          <DashboardWidget />
        </ErrorBoundary>
        <ErrorBoundary fallback={<WidgetError />}>
          <AnalyticsWidget />
        </ErrorBoundary>
      </main>
      <Footer />
    </ErrorBoundary>
  );
}
```

## What Error Boundaries Catch

✅ **Catches:**
- Errors during rendering
- Errors in lifecycle methods
- Errors in constructors of child components

❌ **Does NOT catch:**
- Event handler errors
- Async code (setTimeout, promises)
- Server-side rendering errors
- Errors in the error boundary itself

## Handling Event Errors

```tsx
function Button() {
  const handleClick = () => {
    try {
      doSomethingRisky();
    } catch (error) {
      // Handle error in event handler
      showErrorToast(error.message);
    }
  };
  
  return <button onClick={handleClick}>Click</button>;
}
```

## Using react-error-boundary Library

For a more ergonomic API:

```tsx
import { ErrorBoundary, useErrorBoundary } from 'react-error-boundary';

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
      onError={(error, info) => logError(error, info)}
      onReset={() => {
        // Reset app state
      }}
    >
      <MyApp />
    </ErrorBoundary>
  );
}

// Trigger error boundary from event handlers
function DataLoader() {
  const { showBoundary } = useErrorBoundary();
  
  const loadData = async () => {
    try {
      await fetchData();
    } catch (error) {
      showBoundary(error); // Triggers nearest error boundary
    }
  };
  
  return <button onClick={loadData}>Load</button>;
}
```

## Resources

- [Error Boundaries](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) — Official React documentation on Error Boundaries

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*