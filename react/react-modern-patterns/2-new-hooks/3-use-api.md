---
source_course: "react-modern-patterns"
source_lesson: "react-use-api"
---

# The use API

## Introduction

React 19 introduces `use`, a new API that breaks one of the oldest rules of hooks: it can be called conditionally. Unlike `useContext` or `useState`, `use` can appear inside `if` statements, loops, and early returns. It serves two primary purposes: reading the value of a Promise (integrating with Suspense) and reading Context values conditionally. This makes it one of the most flexible tools in the React 19 toolkit.

## Key Concepts

- **`use(promise)`**: Reads the resolved value of a Promise. The component suspends (shows a Suspense fallback) until the Promise resolves.
- **`use(context)`**: Reads a Context value, just like `useContext`, but with the ability to be called conditionally.
- **Suspense Integration**: When `use` encounters an unresolved Promise, it triggers the nearest `<Suspense>` boundary's fallback.
- **Conditional Calling**: Unlike traditional hooks, `use` can be placed inside `if` blocks, `for` loops, and after early returns.

## Real World Context

Imagine a dashboard component that only shows a user's premium content if they are logged in. With `useContext`, you could not place the context read after an early return. With `use`, you can check a condition first and only read the context when needed. For Promise reading, consider a product page where the product data is fetched as a Promise — `use` lets the component suspend until the data is ready, and Suspense shows a skeleton loader in the meantime.

## Deep Dive

Reading a Promise with `use`:

```tsx
import { use, Suspense } from "react";

function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);

  return (
    <div>
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.bio}</p>
    </div>
  );
}

function App({ userId }: { userId: string }) {
  // Important: create Promise outside render or cache it
  const userPromise = fetchUser(userId);

  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

Reading Context conditionally:

```tsx
import { use } from "react";
import { ThemeContext } from "./contexts";

function OptionalTheme({ useTheme, children }: { useTheme: boolean; children: React.ReactNode }) {
  if (!useTheme) {
    return <div className="default">{children}</div>;
  }

  // This is valid! use() can be called after an early return
  const theme = use(ThemeContext);

  return (
    <div style={{ background: theme.bg, color: theme.text }}>
      {children}
    </div>
  );
}
```

An important detail: when using `use` with Promises, the Promise should not be created during render. If you write `use(fetch(...))` directly inside a component, a new Promise is created on every render, causing an infinite Suspense loop. Instead, create the Promise in a parent component, a loader, or use a caching layer.

## Common Pitfalls

1. **Creating Promises during render** — Writing `use(fetchData())` inside the component body creates a new Promise every render, causing infinite suspension. Always lift Promise creation outside the component or use a cache.
2. **Forgetting the Suspense boundary** — When using `use` with a Promise, there must be a `<Suspense>` ancestor. Without one, React will throw an error because it has nowhere to show a fallback.

## Best Practices

1. **Cache or hoist Promises** — Create Promises in route loaders, parent components, or use a caching mechanism so the same Promise reference is reused across renders.
2. **Use `use(Context)` for conditional reads** — If you have components that only need context in certain branches, prefer `use(Context)` over `useContext(Context)` to avoid unnecessary context subscriptions.

## Summary

- `use` can read Promises (triggering Suspense) and Context values, and unlike hooks it can be called conditionally.
- Promises passed to `use` must be cached or created outside render to avoid infinite suspension loops.
- `use(Context)` is a drop-in replacement for `useContext(Context)` with the added benefit of conditional calling.

## Code Examples

**use API for reading Promises with Suspense and conditional Context**

```tsx
import { use, Suspense } from "react";
import { ThemeContext } from "./contexts";

// Reading a Promise — suspends until resolved
function UserProfile({ userPromise }: { userPromise: Promise<User> }) {
  const user = use(userPromise);
  return <h2>{user.name}</h2>;
}

// Reading Context conditionally — not possible with useContext
function ConditionalTheme({ enabled }: { enabled: boolean }) {
  if (!enabled) return <div>No theme</div>;

  const theme = use(ThemeContext); // Valid after early return!
  return <div style={{ color: theme.primary }}>Themed!</div>;
}

// Usage
function App() {
  const userPromise = fetchUser("123");
  return (
    <Suspense fallback={<p>Loading...</p>}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```


## Resources

- [use Reference](https://react.dev/reference/react/use) — Official use API documentation

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*