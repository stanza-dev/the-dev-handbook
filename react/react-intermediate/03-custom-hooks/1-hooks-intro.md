---
source_course: "react-intermediate"
source_lesson: "react-custom-hooks-intro"
---

# Creating Custom Hooks

## Introduction

Custom hooks let you extract stateful logic from components into reusable functions. If you find yourself duplicating the same `useState` + `useEffect` patterns across multiple components, a custom hook is the right abstraction. Custom hooks share logic (not state) — each component that calls a custom hook gets its own independent copy of the state.

## Key Concepts

- **Custom Hook**: A JavaScript function whose name starts with `use` and that calls other hooks internally.
- **Logic Reuse**: Custom hooks extract reusable stateful logic without changing the component hierarchy.
- **Independent State**: Each component calling the same custom hook gets its own isolated state instance.
- **Composition**: Custom hooks can call other custom hooks, building complex behavior from simple pieces.

## Real World Context

Every application needs common behaviors: debouncing user input, persisting state to localStorage, tracking window dimensions, managing form validation, or fetching data with loading and error states. Instead of reimplementing these patterns in every component, you write them once as custom hooks and import them wherever needed. Libraries like TanStack Query, React Hook Form, and use-debounce are essentially collections of well-tested custom hooks.

## Deep Dive

### Naming Convention

Custom hooks MUST start with `use`. This is not just a convention — React's linter uses it to enforce the Rules of Hooks (no conditional calls, no calls inside loops):

```tsx
// Good: starts with 'use'
function useLocalStorage<T>(key: string, initialValue: T) { ... }
function useDebounce<T>(value: T, delay: number): T { ... }
function useToggle(initial?: boolean) { ... }

// Bad: not recognized as a hook
function getLocalStorage(key: string) { ... } // Linter won't check hook rules
```

### Building a useLocalStorage Hook

```tsx
import { useState, useEffect } from 'react';

function useLocalStorage<T>(key: string, initialValue: T) {
  const [value, setValue] = useState<T>(() => {
    try {
      const stored = localStorage.getItem(key);
      return stored ? JSON.parse(stored) : initialValue;
    } catch {
      return initialValue;
    }
  });

  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]);

  return [value, setValue] as const;
}

// Usage in different components — each gets independent state
function ThemeSettings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  return <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>{theme}</button>;
}

function LanguageSettings() {
  const [lang, setLang] = useLocalStorage('language', 'en');
  return <select value={lang} onChange={e => setLang(e.target.value)}>...</select>;
}
```

### Building a useDebounce Hook

```tsx
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchBar() {
  const [query, setQuery] = useState('');
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery) {
      searchApi(debouncedQuery);
    }
  }, [debouncedQuery]);

  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}
```

### Return Value Patterns

Custom hooks can return arrays (like useState), objects, or single values:

```tsx
// Array return (positional, rename-friendly)
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  const toggle = useCallback(() => setValue(v => !v), []);
  return [value, toggle] as const;
}

// Object return (named, self-documenting)
function useFetch<T>(url: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  // ... fetch logic ...
  return { data, loading, error };
}
```

## Common Pitfalls

1. **Thinking custom hooks share state between components** — Each component that calls `useLocalStorage('theme')` gets its own independent state. The hook shares logic, not data. To share data, use Context.
2. **Creating hooks that are too specific** — A hook called `useUserProfileFormValidation` is unlikely to be reusable. Keep hooks general: `useFormValidation`, `useAsync`, `useDebounce`.

## Best Practices

1. **Start by extracting from a component** — Write the logic in a component first, then extract it into a hook when you need it in a second place. Do not create abstract hooks before you have concrete use cases.
2. **Return a clean, minimal API** — Only expose what consumers need. Hide internal state and helper functions that are implementation details.

## Summary

- Custom hooks are functions starting with `use` that encapsulate reusable stateful logic.
- Each component calling a custom hook gets its own independent copy of the state.
- Return arrays for positional values (like useState) and objects for named values (like useFetch).

## Code Examples

**A useWindowSize custom hook for responsive layouts**

```tsx
import { useState, useEffect } from 'react';

function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// Usage
function ResponsiveLayout() {
  const { width } = useWindowSize();
  return width < 768 ? <MobileNav /> : <DesktopNav />;
}
```


## Resources

- [Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks) — Official guide to custom hooks

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*