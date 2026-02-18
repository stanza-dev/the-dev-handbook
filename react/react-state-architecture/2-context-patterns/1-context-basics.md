---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-context-basics"
---

# Context API Fundamentals

Context provides a way to pass data through the component tree without prop drilling.

## Creating Context

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Create context with default value
const ThemeContext = createContext(null);

// 2. Create Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const value = {
    theme,
    toggleTheme: () => setTheme(t => t === 'light' ? 'dark' : 'light'),
  };
  
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// 3. Create custom hook
function useTheme() {
  const context = useContext(ThemeContext);
  if (context === null) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

export { ThemeProvider, useTheme };
```

## Using Context

```jsx
// Provider at app level
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}

// Consumer anywhere in tree
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      Current: {theme}
    </button>
  );
}
```

## TypeScript with Context

```tsx
type Theme = 'light' | 'dark';

type ThemeContextType = {
  theme: Theme;
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType | null>(null);

function useTheme(): ThemeContextType {
  const context = useContext(ThemeContext);
  if (context === null) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}
```

## Multiple Contexts

```jsx
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <CartProvider>
          <Router />
        </CartProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}
```

## Context with Default Value

```jsx
// With meaningful default
const LocaleContext = createContext('en-US');

// Consumer works even without Provider
function DateDisplay({ date }) {
  const locale = useContext(LocaleContext);
  return <span>{date.toLocaleDateString(locale)}</span>;
}
```

## When to Use Context

✅ **Good use cases**:
- Theme (dark/light mode)
- Authentication/user data
- Locale/internationalization
- Feature flags
- Toast/notification system

❌ **Avoid for**:
- Frequently changing data (every keystroke)
- Large data sets
- When prop drilling is only 2-3 levels

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*