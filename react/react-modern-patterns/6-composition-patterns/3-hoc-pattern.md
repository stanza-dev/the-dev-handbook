---
source_course: "react-modern-patterns"
source_lesson: "react-hoc-pattern"
---

# Higher-Order Components (HOC)

## Introduction

A Higher-Order Component is a function that takes a component and returns a new component with enhanced behavior. HOCs are a pattern for reusing component logic that predates hooks. While hooks have replaced many HOC use cases, understanding HOCs is essential for working with existing codebases and libraries that still use them.

## Key Concepts

- **Higher-Order Component (HOC)**: A function that accepts a component and returns a new component wrapping the original with additional behavior or data.
- **Cross-cutting concerns**: Features that affect many components, such as logging, authentication, or theming. HOCs are designed to address these.
- **Convention over implementation**: HOCs follow naming conventions like `withAuth`, `withTheme`, `withRouter` to indicate the enhancement they provide.

## Real World Context

React Router's `withRouter`, Redux's `connect`, and Relay's container components are all HOCs. A `withAuth` HOC wraps a page component, checks authentication status, and either renders the page or redirects to login. This keeps auth logic centralized instead of duplicated in every protected page.

## Deep Dive

A basic HOC wraps a component and injects additional props:

```tsx
function withLogger<T extends object>(WrappedComponent: React.ComponentType<T>) {
  return function LoggedComponent(props: T) {
    useEffect(() => {
      console.log(`${WrappedComponent.displayName || 'Component'} mounted`);
    }, []);
    return <WrappedComponent {...props} />;
  };
}

const LoggedButton = withLogger(Button);
```

A more practical example is a `withAuth` HOC:

```tsx
function withAuth<T extends object>(WrappedComponent: React.ComponentType<T>) {
  return function AuthenticatedComponent(props: T) {
    const { user, isLoading } = useAuth();

    if (isLoading) return <LoadingScreen />;
    if (!user) return <Navigate to="/login" />;

    return <WrappedComponent {...props} />;
  };
}

const ProtectedDashboard = withAuth(Dashboard);
```

HOCs can be composed together:

```tsx
const EnhancedPage = withAuth(withTheme(withAnalytics(PageComponent)));
```

**When to use HOCs vs hooks**: Hooks are preferred for new code because they are simpler, more composable, and do not create extra wrapper components. However, HOCs remain useful when you need to wrap a component with additional JSX (conditional rendering, providers) or when working with class components.

## Common Pitfalls

1. **Losing the display name** — HOCs create wrapper components that obscure the original in React DevTools. Always set `displayName` on the returned component.
2. **Prop name collisions** — If the HOC injects a prop with the same name as one the wrapped component expects, it causes bugs. Use namespacing or unique prop names.

## Best Practices

1. **Prefer hooks for new code** — Custom hooks are simpler and avoid wrapper component nesting. Use HOCs only when hooks cannot solve the problem.
2. **Set displayName for debugging** — Assign a descriptive displayName to the wrapper component for clear React DevTools output.

## Summary

- HOCs are functions that wrap components to add behavior, following the `withXxx` naming convention.
- Hooks have replaced most HOC use cases, but HOCs remain in many libraries and legacy codebases.
- When using HOCs, set displayName and watch for prop name collisions.

## Code Examples

**Higher-Order Component for authentication**

```tsx
function withAuth<T extends object>(Component: React.ComponentType<T>) {
  function AuthenticatedComponent(props: T) {
    const { user, isLoading } = useAuth();
    if (isLoading) return <Spinner />;
    if (!user) return <Navigate to="/login" />;
    return <Component {...props} />;
  }
  AuthenticatedComponent.displayName = `withAuth(${Component.displayName || Component.name})`;
  return AuthenticatedComponent;
}

const ProtectedDashboard = withAuth(Dashboard);
```


## Resources

- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) — Official React guide on the modern alternative to HOCs

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*