---
source_course: "react"
source_lesson: "react-context-basics"
---

# Context API

Context enables data sharing across your component tree without manually threading props through every level.

## Ideal Use Cases

- UI themes (dark/light mode)
- Authenticated user data
- Language/locale preferences
- Application-wide settings

## The Pattern

1. **Create** - Define a context with `createContext`
2. **Provide** - Wrap components with the context
3. **Consume** - Access values with `useContext`

## React 19 Syntax

Use the context directly as a JSX element (no `.Provider` needed):

```jsx
<ThemeContext value={theme}>
  {children}
</ThemeContext>
```

## Code Examples

**Complete Context pattern with React 19 syntax**

```tsx
import { createContext, useContext, useState } from 'react';

// 1. Create context with sensible default
type Theme = 'light' | 'dark';
type ThemeContextType = {
  theme: Theme;
  toggleTheme: () => void;
};

const ThemeContext = createContext<ThemeContextType | null>(null);

// 2. Provider component with state logic
function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<Theme>('light');
  
  const toggleTheme = () => {
    setTheme(current => current === 'light' ? 'dark' : 'light');
  };
  
  // React 19: Use context directly as JSX element
  return (
    <ThemeContext value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext>
  );
}

// 3. Custom hook for safe consumption
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Usage in components
function ThemeToggle() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button onClick={toggleTheme}>
      Switch to {theme === 'light' ? '🌙 Dark' : '☀️ Light'}
    </button>
  );
}

// App setup
function App() {
  return (
    <ThemeProvider>
      <Header />
      <Main />
    </ThemeProvider>
  );
}
```


## Resources

- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context) — Official Context guide

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*