---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-context-performance"
---

# Context Performance Patterns

Avoid unnecessary re-renders with proper Context patterns.

## The Problem: Re-renders

When context value changes, ALL consumers re-render:

```jsx
// ❌ Bad: New object every render
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  // This creates new object every render!
  return (
    <AppContext.Provider value={{ user, setUser, theme, setTheme }}>
      {children}
    </AppContext.Provider>
  );
}
```

## Solution 1: useMemo for Value

```jsx
function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  
  const value = useMemo(() => ({
    user,
    setUser,
    theme,
    setTheme,
  }), [user, theme]);
  
  return (
    <AppContext.Provider value={value}>
      {children}
    </AppContext.Provider>
  );
}
```

## Solution 2: Split Contexts

```jsx
// Separate contexts for different concerns
const UserContext = createContext(null);
const ThemeContext = createContext(null);

function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const value = useMemo(() => ({ user, setUser }), [user]);
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const value = useMemo(() => ({ theme, setTheme }), [theme]);
  return (
    <ThemeContext.Provider value={value}>
      {children}
    </ThemeContext.Provider>
  );
}

// Now theme changes don't re-render user consumers!
```

## Solution 3: Separate State and Dispatch

```jsx
const StateContext = createContext(null);
const DispatchContext = createContext(null);

function AppProvider({ children }) {
  const [state, dispatch] = useReducer(reducer, initialState);
  
  return (
    <StateContext.Provider value={state}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Components needing only actions don't re-render on state change
function AddButton() {
  const dispatch = useContext(DispatchContext);
  return <button onClick={() => dispatch({ type: 'add' })}>Add</button>;
}
```

## Solution 4: Select Pattern

```jsx
// Create selector hook
function useUser() {
  const { user } = useContext(AppContext);
  return user;
}

function useTheme() {
  const { theme } = useContext(AppContext);
  return theme;
}

// Better: Use a state management library with selectors
import { create } from 'zustand';
import { useShallow } from 'zustand/react/shallow';

const useStore = create((set) => ({
  user: null,
  theme: 'light',
  setUser: (user) => set({ user }),
  setTheme: (theme) => set({ theme }),
}));

// Only re-renders when user changes
function UserDisplay() {
  const user = useStore((state) => state.user);
  return <span>{user?.name}</span>;
}
```

## Context Value Stability

```jsx
function Provider({ children }) {
  const [count, setCount] = useState(0);
  
  // ❌ Bad: Creates new function every render
  const increment = () => setCount(c => c + 1);
  
  // ✅ Good: Stable function reference
  const increment = useCallback(() => setCount(c => c + 1), []);
  
  const value = useMemo(() => ({
    count,
    increment,
  }), [count, increment]);
  
  return (
    <Context.Provider value={value}>
      {children}
    </Context.Provider>
  );
}
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*