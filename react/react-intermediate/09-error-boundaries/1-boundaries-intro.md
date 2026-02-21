---
source_course: "react-intermediate"
source_lesson: "react-error-boundaries-intro"
---

# Introduction to Error Boundaries

## Introduction

Error boundaries are React components that catch JavaScript errors anywhere in their child component tree, log those errors, and display a fallback UI instead of crashing the entire application. They act as a safety net, ensuring that a bug in one isolated widget does not bring down your whole page.

## Key Concepts

- **Error Boundary**: A class component that implements `static getDerivedStateFromError()` and/or `componentDidCatch()` to catch rendering errors in its subtree.
- **Fallback UI**: The alternative interface shown when an error is caught, replacing the broken component tree.
- **Component Stack**: The trace React provides showing exactly which components led to the error.

## Real World Context

Imagine a dashboard with multiple widgets: a chart, a notifications panel, and a user profile card. Without error boundaries, a single bug in the chart component would crash the entire dashboard. With error boundaries wrapping each widget, only the chart shows an error message while the rest of the dashboard continues working normally.

## Deep Dive

React introduced error boundaries in version 16 as a response to the problem of corrupted UI. Before error boundaries, a JavaScript error inside a component would leave React in a broken state, often rendering a blank screen with no useful information for the user.

Error boundaries work during the React render phase and lifecycle methods. They catch errors that occur during rendering, in lifecycle methods, and in constructors of the whole tree below them.

```tsx
import { Component, ReactNode, ErrorInfo } from 'react';

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

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div role="alert">
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
```

However, error boundaries do **not** catch errors in event handlers, asynchronous code such as `setTimeout` or promise callbacks, server-side rendering, or errors thrown in the error boundary itself.

## Common Pitfalls

1. **Assuming all errors are caught** — Event handler errors, async code errors, and errors in the boundary itself are not caught. Use `try/catch` for those scenarios.
2. **Placing a single boundary at the root** — This provides no isolation. If one component fails, the entire app shows the fallback.

## Best Practices

1. **Use multiple boundaries at strategic levels** — Wrap independent features separately so a failure in one does not affect others.
2. **Provide meaningful fallback UIs** — Match the fallback complexity to the component it replaces, and offer retry options when possible.

## Summary

- Error boundaries catch rendering errors in their child tree and display a fallback UI.
- They must be class components implementing `getDerivedStateFromError` or `componentDidCatch`.
- Place them strategically around independent features for maximum resilience.

## Code Examples

**Basic error boundary class component with fallback**

```tsx
import { Component, ReactNode, ErrorInfo } from 'react';

type Props = { children: ReactNode; fallback?: ReactNode };
type State = { hasError: boolean; error?: Error };

class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: ErrorInfo) {
    console.error('Boundary caught:', error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <h2>Something went wrong</h2>;
    }
    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary fallback={<div>Widget failed</div>}>
      <MyWidget />
    </ErrorBoundary>
  );
}
```


## Resources

- [Catching rendering errors with an error boundary](https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary) — Official React documentation on error boundaries

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*