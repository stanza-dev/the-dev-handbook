---
source_course: "react-intermediate"
source_lesson: "react-performance-patterns"
---

# Advanced Performance Patterns

## Introduction

Beyond basic memoization, architectural patterns can prevent performance issues from occurring in the first place.

## Key Concepts

**Composition patterns** restructure component trees to naturally avoid unnecessary renders. **State colocation** keeps state close to where it's used. **Selective subscriptions** minimize what triggers updates.

## Real World Context

These patterns are used in:
- Large-scale applications with complex state
- Real-time dashboards with frequent updates
- Applications where memoization isn't enough
- Preventing performance issues by design

## Deep Dive

### State Colocation

Move state down to where it's used:

```tsx
// ❌ State too high - entire app re-renders on hover
function App() {
  const [hoveredId, setHoveredId] = useState(null);
  return (
    <div>
      <Header />
      <Sidebar />
      <ItemList hoveredId={hoveredId} onHover={setHoveredId} />
    </div>
  );
}

// ✅ State colocated - only ItemList re-renders
function App() {
  return (
    <div>
      <Header />
      <Sidebar />
      <ItemList /> {/* Manages its own hover state */}
    </div>
  );
}
```

### Children as Props Pattern

Pass children to avoid re-renders:

```tsx
// ❌ Children re-render when ExpensiveParent state changes
function ExpensiveParent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      <ExpensiveChild /> {/* Re-renders! */}
    </div>
  );
}

// ✅ Pass children - they don't re-render
function App() {
  return (
    <ExpensiveParentWrapper>
      <ExpensiveChild /> {/* Stable - doesn't re-render */}
    </ExpensiveParentWrapper>
  );
}

function ExpensiveParentWrapper({ children }) {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Count: {count}</button>
      {children} {/* Same reference - no re-render */}
    </div>
  );
}
```

### Split Context by Update Frequency

```tsx
// ❌ One context - all consumers re-render on any change
const AppContext = createContext({ user: null, theme: 'light', notifications: [] });

// ✅ Split by update frequency
const UserContext = createContext(null); // Rarely changes
const ThemeContext = createContext('light'); // Rarely changes  
const NotificationsContext = createContext([]); // Frequently changes
```

### Selector Pattern for Context

```tsx
// Only re-render when specific value changes
function useContextSelector<T, S>(context: Context<T>, selector: (v: T) => S): S {
  const value = useContext(context);
  const selectedRef = useRef<S>();
  const selected = selector(value);
  
  if (!Object.is(selectedRef.current, selected)) {
    selectedRef.current = selected;
  }
  
  return selectedRef.current as S;
}

// Usage
function UserName() {
  // Only re-renders when user.name changes
  const name = useContextSelector(UserContext, (ctx) => ctx.user?.name);
  return <span>{name}</span>;
}
```

### Component Splitting

```tsx
// ❌ Expensive component mixed with frequently updating state
function Dashboard() {
  const [time, setTime] = useState(new Date());
  useEffect(() => {
    const id = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(id);
  }, []);
  
  return (
    <div>
      <Clock time={time} />
      <ExpensiveChart data={data} /> {/* Re-renders every second! */}
    </div>
  );
}

// ✅ Split into separate components
function Dashboard() {
  return (
    <div>
      <LiveClock /> {/* Contains its own state */}
      <ExpensiveChart data={data} />
    </div>
  );
}

function LiveClock() {
  const [time, setTime] = useState(new Date());
  useEffect(() => {
    const id = setInterval(() => setTime(new Date()), 1000);
    return () => clearInterval(id);
  }, []);
  return <Clock time={time} />;
}
```

## Common Pitfalls

1. **Global state for local concerns**: Not everything needs to be in Redux/Context.
2. **Single monolithic context**: Split contexts by update frequency.
3. **Prop drilling avoidance at all costs**: Sometimes props are simpler than context.

## Best Practices

- Colocate state - keep it as close to usage as possible
- Split components that update at different rates
- Use composition (children prop) to prevent cascading updates
- Split contexts by how often values change
- Consider external state libraries (Zustand, Jotai) with built-in selectors
- Profile to verify patterns actually help

## Summary

Architectural patterns like state colocation, children-as-props, and context splitting prevent performance issues by design. These complement memoization and can be more effective for systemic issues.

## Code Examples

**Advanced performance patterns with context splitting and composition**

```tsx
import { createContext, useContext, useState, ReactNode, memo } from 'react';

// Pattern: Split providers by update frequency
type AuthState = { user: User | null; isAuthenticated: boolean };
type AuthActions = { login: (user: User) => void; logout: () => void };

const AuthStateContext = createContext<AuthState | null>(null);
const AuthActionsContext = createContext<AuthActions | null>(null);

function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  
  // Actions never change - stable reference
  const actions = useMemo(() => ({
    login: (user: User) => setUser(user),
    logout: () => setUser(null)
  }), []);
  
  const state = useMemo(() => ({
    user,
    isAuthenticated: user !== null
  }), [user]);
  
  return (
    <AuthActionsContext.Provider value={actions}>
      <AuthStateContext.Provider value={state}>
        {children}
      </AuthStateContext.Provider>
    </AuthActionsContext.Provider>
  );
}

// Components can subscribe to just what they need
function useAuthState() {
  const ctx = useContext(AuthStateContext);
  if (!ctx) throw new Error('Missing AuthProvider');
  return ctx;
}

function useAuthActions() {
  const ctx = useContext(AuthActionsContext);
  if (!ctx) throw new Error('Missing AuthProvider');
  return ctx;
}

// This component only re-renders when user state changes
function UserGreeting() {
  const { user, isAuthenticated } = useAuthState();
  return isAuthenticated ? <span>Hello, {user?.name}</span> : null;
}

// This component NEVER re-renders from auth changes
// (actions reference is stable)
const LogoutButton = memo(function LogoutButton() {
  const { logout } = useAuthActions();
  return <button onClick={logout}>Logout</button>;
});

// Pattern: Children to prevent re-renders
function ExpandableSection({ title, children }: {
  title: string;
  children: ReactNode;
}) {
  const [isExpanded, setIsExpanded] = useState(false);
  
  return (
    <div>
      <button onClick={() => setIsExpanded(e => !e)}>
        {title} {isExpanded ? '▼' : '▶'}
      </button>
      {isExpanded && children}
    </div>
  );
}

// Usage - ExpensiveContent doesn't re-render on expand/collapse toggle
<ExpandableSection title="Details">
  <ExpensiveContent /> {/* Stable reference */}
</ExpandableSection>
```


## Resources

- [Before You memo()](https://overreacted.io/before-you-memo/) — Dan Abramov on composition patterns for performance

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*