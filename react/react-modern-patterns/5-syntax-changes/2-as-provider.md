---
source_course: "react-modern-patterns"
source_lesson: "react-context-as-provider"
---

# Context as Provider

## Introduction

React 19 simplifies how you provide context values to your component tree. Instead of writing `<ThemeContext.Provider value={theme}>`, you can now write `<ThemeContext value={theme}>` — using the context object itself as the JSX element. This might seem like a small change, but it reduces visual noise, especially in applications that nest multiple providers, and it makes the relationship between context creation and provision more intuitive.

## Key Concepts

- **Direct context rendering**: In React 19, you render the context object itself as a JSX element with a `value` prop, eliminating the `.Provider` suffix.
- **Backward compatibility**: `<Context.Provider value={...}>` still works in React 19, so existing code does not break.
- **Deprecation path**: The `.Provider` pattern will be deprecated in a future React version.
- **Nested providers**: The simplified syntax makes deeply nested provider trees easier to read.

## Real World Context

Many React applications wrap their root component in a "provider tower" — five or more nested context providers for themes, authentication, routing, feature flags, and notifications. With the old syntax, each provider adds `.Provider` to the nesting, making the JSX harder to scan. The new syntax shaves off that suffix from every level, producing cleaner code that is easier to maintain.

## Deep Dive

Here is the old versus new syntax:

```tsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext("light");
const AuthContext = createContext<User | null>(null);
const FeatureFlagContext = createContext<Set<string>>(new Set());

// React 18 — verbose provider tower
function OldProviders({ children }: { children: React.ReactNode }) {
  return (
    <ThemeContext.Provider value="dark">
      <AuthContext.Provider value={currentUser}>
        <FeatureFlagContext.Provider value={enabledFlags}>
          {children}
        </FeatureFlagContext.Provider>
      </AuthContext.Provider>
    </ThemeContext.Provider>
  );
}

// React 19 — cleaner syntax
function NewProviders({ children }: { children: React.ReactNode }) {
  return (
    <ThemeContext value="dark">
      <AuthContext value={currentUser}>
        <FeatureFlagContext value={enabledFlags}>
          {children}
        </FeatureFlagContext>
      </AuthContext>
    </ThemeContext>
  );
}
```

Consumers work exactly the same way regardless of which syntax the provider uses:

```tsx
function ThemedButton() {
  const theme = useContext(ThemeContext);
  // or with the use API:
  // const theme = use(ThemeContext);

  return (
    <button className={theme === "dark" ? "btn-dark" : "btn-light"}>
      Current theme: {theme}
    </button>
  );
}
```

You can override context in a subtree using the same clean syntax:

```tsx
function LightSection({ children }: { children: React.ReactNode }) {
  return (
    <ThemeContext value="light">
      {children}
    </ThemeContext>
  );
}
```

## Common Pitfalls

1. **Mixing `.Provider` and direct syntax in the same codebase** — While both work, inconsistency can confuse team members. Agree on one style during migration.
2. **Forgetting the `value` prop** — Both the old and new syntax require the `value` prop. Rendering `<ThemeContext>` without `value` will pass `undefined`, overriding the default value from `createContext`.

## Best Practices

1. **Adopt the new syntax in new code** — Use `<Context value={...}>` for all new providers. Migrate existing `.Provider` usage incrementally.
2. **Combine with the `use` API for reading** — Pair the new provider syntax with `use(Context)` for conditional context reading, completing the modernization of your context patterns.

## Summary

- React 19 lets you use the context object directly as a JSX element: `<ThemeContext value={theme}>` instead of `<ThemeContext.Provider value={theme}>`.
- The old `.Provider` syntax still works but will be deprecated in a future version.
- Consumer patterns (`useContext` and `use`) are unchanged and work with both provider syntaxes.

## Code Examples

**Context as provider without .Provider suffix in React 19**

```tsx
import { createContext, useContext, useState } from "react";

const ThemeContext = createContext("light");
const UserContext = createContext<{ name: string } | null>(null);

// React 19 — clean provider syntax
function App({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState("dark");
  const [user] = useState({ name: "Alice" });

  return (
    <ThemeContext value={theme}>
      <UserContext value={user}>
        {children}
      </UserContext>
    </ThemeContext>
  );
}

// Consumer — unchanged
function Header() {
  const theme = useContext(ThemeContext);
  const user = useContext(UserContext);
  return <h1 className={theme}>Welcome, {user?.name}</h1>;
}
```


## Resources

- [createContext Reference](https://react.dev/reference/react/createContext) — Official createContext documentation with new provider syntax

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*