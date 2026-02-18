---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-deep-usecontext"
---

# useContext: Global State Without Prop Drilling

`useContext` lets you read and subscribe to context from your component. Context is designed to share data that can be considered "global" for a tree of React components.

## The Three-Step Pattern

### 1. Create the Context

```tsx
import { createContext } from 'react';

type Theme = 'light' | 'dark';

type ThemeContextType = {
  theme: Theme;
  toggleTheme: () => void;
};

// Provide a meaningful default or null
const ThemeContext = createContext<ThemeContextType | null>(null);
```

### 2. Provide the Context

```tsx
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');

  const toggleTheme = useCallback(() => {
    setTheme((prev) => (prev === 'light' ? 'dark' : 'light'));
  }, []);

  // Memoize the value to prevent unnecessary re-renders
  const value = useMemo(
    () => ({ theme, toggleTheme }),
    [theme, toggleTheme]
  );

  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### 3. Consume with a Custom Hook

```tsx
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === null) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

// Usage in any component
function Header() {
  const { theme, toggleTheme } = useTheme();
  return (
    <header className={theme}>
      <button onClick={toggleTheme}>Toggle Theme</button>
    </header>
  );
}
```

## React 19 Simplified Syntax

In React 19, you can use the Context directly as a provider:

```tsx
// Before (still works)
<ThemeContext.Provider value={value}>
  {children}
</ThemeContext.Provider>

// React 19 - simplified
<ThemeContext value={value}>
  {children}
</ThemeContext>
```

## When to Use Context

✅ **Good use cases:**
- Theme data (dark/light mode)
- Current user/authentication
- Locale/language preferences
- Feature flags

❌ **Avoid for:**
- Frequently changing data (causes many re-renders)
- Data only needed by a few nearby components (use props)
- Complex state logic (combine with useReducer or external libraries)

## Resources

- [useContext API Reference](https://react.dev/reference/react/useContext) — Official React documentation for useContext hook
- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context) — Learn guide on using Context effectively

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*