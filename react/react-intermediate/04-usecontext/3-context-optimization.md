---
source_course: "react-intermediate"
source_lesson: "react-context-optimization"
---

# Context Performance Optimization

## Introduction

Context is a powerful tool, but it has a significant performance characteristic: when a context value changes, ALL components that consume that context re-render. For contexts that change infrequently (theme, auth), this is fine. For contexts with complex or frequently changing values, you need optimization strategies to prevent cascading re-renders across your application.

## Key Concepts

- **Consumer Re-rendering**: Every component calling `useContext(SomeContext)` re-renders when the context value changes, regardless of which part of the value it uses.
- **Value Memoization**: Using `useMemo` on the value object to prevent re-renders caused by the provider's parent re-rendering.
- **Context Splitting**: Breaking one large context into multiple smaller, focused contexts.
- **State Colocation**: Keeping state as close to where it is used as possible, avoiding context for state that only a few nearby components need.

## Real World Context

An e-commerce app has a global context with the user profile, shopping cart, and UI preferences. Every time the user adds an item to the cart, the cart count changes, the context value changes, and every component consuming this context re-renders — including the user profile sidebar and the theme toggle, which have nothing to do with the cart. Splitting the context or optimizing the value structure eliminates these wasted renders.

## Deep Dive

### Problem: Unnecessary Re-renders

```tsx
// Bad: One big context — all consumers re-render on any change
const AppContext = createContext({
  user: null,
  cart: { items: [], total: 0 },
  theme: 'light',
  notifications: [],
});

function CartIcon() {
  const { cart } = useContext(AppContext);
  // Re-renders when theme changes, when notifications change, etc.
  return <span>{cart.items.length}</span>;
}
```

### Solution 1: Split Contexts

```tsx
// Good: Each domain has its own context
const UserContext = createContext<User | null>(null);
const CartContext = createContext<CartState>({ items: [], total: 0 });
const ThemeContext = createContext<Theme>('light');

function CartIcon() {
  const cart = useContext(CartContext);
  // Only re-renders when cart changes — not when theme or user changes
  return <span>{cart.items.length}</span>;
}
```

### Solution 2: Memoize the Provider Value

When the provider component re-renders for reasons unrelated to the context value (e.g., a parent re-renders), the context value should be memoized:

```tsx
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  // Without useMemo: a new object is created on every render,
  // causing all consumers to re-render
  const value = useMemo(() => ({
    theme,
    toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light'),
  }), [theme]);

  return <ThemeContext value={value}>{children}</ThemeContext>;
}
```

### Solution 3: Component Composition Instead of Context

Sometimes you do not need context at all. Passing components as props avoids prop drilling without the re-render cost:

```tsx
// Instead of using context to pass user data deep down...
function Page({ user }: { user: User }) {
  return (
    <Layout
      header={<Header userName={user.name} avatar={user.avatarUrl} />}
      sidebar={<Sidebar role={user.role} />}
    >
      <Content />
    </Layout>
  );
}
```

### Solution 4: Selector Pattern (with useSyncExternalStore)

For advanced cases, you can build a selector-based context that only re-renders consumers when their selected slice changes:

```tsx
function useCartItemCount() {
  const cart = useContext(CartContext);
  // This component still re-renders on any cart change,
  // but wrapping in React.memo with the count check limits child re-renders
  return cart.items.length;
}
```

### When Context Is Not the Right Tool

- **Frequently updating values** (animations, mouse position, scroll) — Use state in the component that needs it.
- **Global state management for complex apps** — Consider Zustand, Jotai, or Redux Toolkit for fine-grained subscriptions.
- **Server state** — Use TanStack Query or SWR instead of putting fetched data in context.

## Common Pitfalls

1. **Creating a new value object on every render** — If the provider's parent re-renders, `value={{ theme, toggle }}` creates a new object each time, re-rendering all consumers even though the data has not changed. Always memoize.
2. **Using context for everything** — Context is not a general-purpose state manager. It lacks fine-grained subscriptions, middleware, and devtools. Use it for truly global, infrequently changing data.

## Best Practices

1. **Start with component composition** — Before reaching for context, try passing components as props (the "slots" pattern). This solves many prop drilling cases without context's re-render trade-offs.
2. **Profile context re-renders** — Use React DevTools "Highlight updates when components render" to see if context changes are causing unnecessary work.

## Summary

- Context re-renders ALL consumers when its value changes, so keep context values focused and infrequently changing.
- Split large contexts into domain-specific ones, memoize provider values, and prefer composition over context for simple cases.
- For high-frequency updates or complex state, use dedicated state management libraries instead of context.

## Code Examples

**Memoizing the context value to prevent unnecessary consumer re-renders**

```tsx
import { createContext, useContext, useMemo, useState } from 'react';

type SettingsContextType = {
  language: string;
  setLanguage: (lang: string) => void;
};

const SettingsContext = createContext<SettingsContextType | null>(null);

function SettingsProvider({ children }: { children: React.ReactNode }) {
  const [language, setLanguage] = useState('en');

  // Memoize to prevent unnecessary consumer re-renders
  const value = useMemo(() => ({ language, setLanguage }), [language]);

  return <SettingsContext value={value}>{children}</SettingsContext>;
}
```


## Resources

- [Optimizing Context Re-renders](https://react.dev/learn/passing-data-deeply-with-context#optimizing-re-renders-when-passing-objects-and-functions) — Official optimization tips for context

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*