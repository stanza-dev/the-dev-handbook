---
source_course: "react-intermediate"
source_lesson: "react-context-basics"
---

# React Context Fundamentals

## Introduction

As React applications grow, you often need to share data across many components at different levels of the tree. Passing props through every intermediate component (known as prop drilling) is tedious and makes code fragile. The Context API lets you broadcast data to any component in the subtree without explicitly threading props through every level.

## Key Concepts

- **Context**: A mechanism for sharing values between components without passing props through every level of the tree.
- **createContext**: A function that creates a context object with a default value.
- **Provider**: A component (or JSX element in React 19) that makes a context value available to its descendants.
- **useContext**: A hook that reads the current value of a context from the nearest provider above in the tree.

## Real World Context

Consider an application with a theme (light/dark mode) that affects every component: headers, buttons, cards, inputs, modals. Without Context, you would pass a `theme` prop through the App to the Layout to the Sidebar to the NavItem to the NavLink — through components that do not even use the theme themselves. With Context, any component anywhere in the tree can access the theme directly.

## Deep Dive

### The Three Steps

**1. Create a Context**

```tsx
import { createContext } from 'react';

type AuthContextType = {
  user: { id: string; name: string } | null;
  login: (credentials: { email: string; password: string }) => Promise<void>;
  logout: () => void;
};

const AuthContext = createContext<AuthContextType | null>(null);
```

**2. Provide the Context**

In React 19, you can use the context directly as a JSX element (no `.Provider` needed):

```tsx
function AuthProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState(null);

  const login = async (credentials: { email: string; password: string }) => {
    const response = await fetch('/api/login', {
      method: 'POST',
      body: JSON.stringify(credentials),
    });
    const userData = await response.json();
    setUser(userData);
  };

  const logout = () => setUser(null);

  return (
    <AuthContext value={{ user, login, logout }}>
      {children}
    </AuthContext>
  );
}
```

**3. Consume the Context**

```tsx
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}

function UserMenu() {
  const { user, logout } = useAuth();

  if (!user) return <LoginButton />;

  return (
    <div>
      <span>Hi, {user.name}!</span>
      <button onClick={logout}>Log out</button>
    </div>
  );
}
```

### React 19 Syntax Change

In previous React versions, you had to use `<AuthContext.Provider value={...}>`. In React 19, the context itself IS the provider:

```tsx
// React 18 and earlier
<AuthContext.Provider value={{ user, login, logout }}>
  {children}
</AuthContext.Provider>

// React 19+
<AuthContext value={{ user, login, logout }}>
  {children}
</AuthContext>
```

### Wiring Up the Provider

Place the provider high enough in the tree that all consumers are descendants:

```tsx
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <Router>
          <Layout />
        </Router>
      </ThemeProvider>
    </AuthProvider>
  );
}
```

## Common Pitfalls

1. **Using Context for frequently changing values** — Context re-renders ALL consumers when the value changes. If you put rapidly changing data (like mouse position or scroll offset) in Context, every consumer re-renders on each change.
2. **Forgetting to check for null** — When the default context value is `null` (as recommended for typed contexts), consumers that render outside a provider will get `null`. Always create a custom hook with a null check.

## Best Practices

1. **Always create a custom hook for consumption** — `useAuth()` is better than `useContext(AuthContext)` because the hook can include the null check and throw a descriptive error.
2. **Use Context for data that changes infrequently** — Theme, locale, authenticated user, and feature flags are ideal. For frequently updated data, consider state management libraries or component-level state.

## Summary

- Context provides a way to pass data through the component tree without prop drilling.
- Create a context with `createContext`, provide it with a Provider component, and consume it with `useContext`.
- In React 19, use the context directly as a JSX element without `.Provider`.

## Code Examples

**Complete Context pattern: create, provide, consume with a custom hook**

```tsx
import { createContext, useContext, useState } from 'react';

type Theme = 'light' | 'dark';
type ThemeContextType = { theme: Theme; toggleTheme: () => void };

const ThemeContext = createContext<ThemeContextType | null>(null);

function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error('useTheme must be used within ThemeProvider');
  return ctx;
}

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  const toggleTheme = () => setTheme(t => t === 'light' ? 'dark' : 'light');

  return (
    <ThemeContext value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext>
  );
}

function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  return <button onClick={toggleTheme}>Current: {theme}</button>;
}
```


## Resources

- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context) — Official Context guide

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*