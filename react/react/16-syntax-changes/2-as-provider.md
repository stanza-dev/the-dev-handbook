---
source_course: "react"
source_lesson: "react-context-as-provider"
---

# Simplified Context Providers

In React 19, use the Context object directly as a provider. No more `.Provider`!

## Before (React 18)

```tsx
<ThemeContext.Provider value={theme}>
  {children}
</ThemeContext.Provider>
```

## After (React 19)

```tsx
<ThemeContext value={theme}>
  {children}
</ThemeContext>
```

## Benefits

- Less boilerplate
- Cleaner JSX
- Easier to read nested providers
- `.Provider` still works for compatibility

## Code Examples

**Context as Provider without .Provider**

```tsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');
const UserContext = createContext(null);

// React 19 - cleaner provider syntax
function App({ children }) {
  const [theme, setTheme] = useState('dark');
  const [user, setUser] = useState({ name: 'Guest' });
  
  return (
    <ThemeContext value={theme}>
      <UserContext value={{ user, setUser }}>
        {children}
      </UserContext>
    </ThemeContext>
  );
}

// Consumer components work the same
function Header() {
  const theme = useContext(ThemeContext);
  const { user } = useContext(UserContext);
  
  return (
    <header className={`header-${theme}`}>
      <h1>Welcome, {user.name}</h1>
    </header>
  );
}

// Override in subtree - same clean syntax
function LightSection({ children }) {
  return (
    <ThemeContext value="light">
      {children}
    </ThemeContext>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*