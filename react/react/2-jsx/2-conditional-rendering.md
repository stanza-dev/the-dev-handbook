---
source_course: "react"
source_lesson: "react-conditional-rendering"
---

# Conditional Rendering

## Introduction

Real-world UIs are rarely static. You need to show or hide elements, switch between views, and display different content based on application state. React handles conditional rendering through standard JavaScript operators, giving you full control over what appears on screen. This lesson covers every pattern you will use in production.

## Key Concepts

- **Ternary Operator**: `condition ? <A /> : <B />` renders one of two branches.
- **Logical AND**: `condition && <Component />` renders the component only when the condition is truthy.
- **Early Return**: Returning `null` or an alternative JSX block early in the function body based on a condition.
- **Variables for JSX**: Assigning JSX to a variable using `if/else` before the return statement.

## Real World Context

Consider a dashboard that shows a loading spinner while data is being fetched, an error message if the request fails, and the actual content when data is ready. You might also have a premium badge that only appears for paid users, and a notification dot that shows only when there are unread messages. Every one of these requires conditional rendering.

## Deep Dive

### Ternary Operator (either/or)

Use ternaries when you always want to render something, but the specific element depends on a condition:

```tsx
function UserGreeting({ isLoggedIn }: { isLoggedIn: boolean }) {
  return (
    <header>
      {isLoggedIn ? (
        <span>Welcome back!</span>
      ) : (
        <button>Sign In</button>
      )}
    </header>
  );
}
```

### Logical AND (show/hide)

Use `&&` when you want to render something or nothing:

```tsx
function NotificationBadge({ count }: { count: number }) {
  return (
    <div className="bell-icon">
      {count > 0 && <span className="badge">{count}</span>}
    </div>
  );
}
```

### Early Return

Use early returns for loading and error states to keep the main render path clean:

```tsx
function UserProfile({ user, isLoading, error }) {
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage message={error.message} />;
  if (!user) return null;

  return (
    <div className="profile">
      <h1>{user.name}</h1>
      <p>{user.bio}</p>
    </div>
  );
}
```

### Watch out for falsy values with &&

A common bug is using a number with `&&`. If the number is `0`, React renders the string `"0"` instead of nothing:

```tsx
// Bug: renders "0" when items is empty
{items.length && <ItemList items={items} />}

// Fix: use an explicit boolean comparison
{items.length > 0 && <ItemList items={items} />}
```

## Common Pitfalls

1. **Rendering `0` with logical AND** — `{count && <Badge />}` renders the number `0` on screen when count is zero because `0` is falsy but still a renderable value in React. Always convert to a boolean: `{count > 0 && <Badge />}`.
2. **Deeply nested ternaries** — Chaining ternaries (`a ? b : c ? d : e`) is hard to read. Extract the logic into a helper function or use early returns instead.

## Best Practices

1. **Use early returns for guard clauses** — Check for loading, error, and empty states at the top of your component. This keeps the happy-path JSX flat and readable.
2. **Prefer ternaries for two-branch rendering and `&&` for show/hide** — Pick the pattern that matches your intent. Do not use `&&` when you need an else branch.

## Summary

- Use the ternary operator for either/or rendering and logical AND for show/hide patterns.
- Always use explicit boolean comparisons with `&&` to avoid accidentally rendering `0` or empty strings.
- Early returns simplify components with multiple conditional states like loading, error, and empty.

## Code Examples

**Multiple conditional rendering patterns in a single component**

```tsx
function Dashboard({ user, notifications }: {
  user: { name: string; isPremium: boolean } | null;
  notifications: number;
}) {
  if (!user) return <LoginPrompt />;

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      {user.isPremium ? (
        <span className="badge-premium">Premium</span>
      ) : (
        <button>Upgrade to Premium</button>
      )}
      {notifications > 0 && (
        <span className="notification-dot">{notifications}</span>
      )}
    </div>
  );
}
```


## Resources

- [Conditional Rendering](https://react.dev/learn/conditional-rendering) — Official guide on conditional rendering patterns

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*