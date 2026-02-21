---
source_course: "react-intermediate"
source_lesson: "react-context-patterns"
---

# Context Patterns

## Introduction

The Context API is flexible, but using it effectively requires understanding common patterns for structuring providers, splitting state from dispatch, and composing multiple contexts. This lesson covers the patterns you will use in production applications to keep your context architecture clean and performant.

## Key Concepts

- **Provider Component Pattern**: Encapsulating the context provider, state, and logic in a dedicated component.
- **Split Contexts**: Separating state from dispatch to reduce unnecessary re-renders.
- **Composable Providers**: Nesting multiple providers to organize different concerns.
- **Default Values**: Providing meaningful defaults versus null with runtime checks.

## Real World Context

A medium-sized application might have contexts for authentication, theme, locale, feature flags, and a shopping cart. Each has its own provider, custom hook, and update logic. Understanding how to structure and compose these contexts prevents spaghetti code and keeps each concern isolated.

## Deep Dive

### The Provider Component Pattern

Encapsulate all context logic in a provider component:

```tsx
const NotificationContext = createContext<{
  notifications: Notification[];
  addNotification: (msg: string, type: 'info' | 'error') => void;
  dismissNotification: (id: string) => void;
} | null>(null);

function NotificationProvider({ children }: { children: React.ReactNode }) {
  const [notifications, setNotifications] = useState<Notification[]>([]);

  const addNotification = useCallback((message: string, type: 'info' | 'error') => {
    const id = crypto.randomUUID();
    setNotifications(prev => [...prev, { id, message, type }]);
    setTimeout(() => dismissNotification(id), 5000);
  }, []);

  const dismissNotification = useCallback((id: string) => {
    setNotifications(prev => prev.filter(n => n.id !== id));
  }, []);

  return (
    <NotificationContext value={{ notifications, addNotification, dismissNotification }}>
      {children}
    </NotificationContext>
  );
}
```

### Split State and Dispatch Contexts

When many components only dispatch actions (without reading state), split the context to prevent unnecessary re-renders:

```tsx
const CartStateContext = createContext<CartState | null>(null);
const CartDispatchContext = createContext<React.Dispatch<CartAction> | null>(null);

function CartProvider({ children }: { children: React.ReactNode }) {
  const [state, dispatch] = useReducer(cartReducer, { items: [], total: 0 });

  return (
    <CartStateContext value={state}>
      <CartDispatchContext value={dispatch}>
        {children}
      </CartDispatchContext>
    </CartStateContext>
  );
}

// Components that only add items use dispatch — won't re-render when cart total changes
function AddToCartButton({ product }) {
  const dispatch = useContext(CartDispatchContext);
  return <button onClick={() => dispatch({ type: 'ADD_ITEM', payload: product })}>Add</button>;
}

// Components that display the cart use state
function CartSummary() {
  const state = useContext(CartStateContext);
  return <span>Total: ${state.total}</span>;
}
```

### Composing Multiple Providers

Create a helper to avoid deeply nested provider trees:

```tsx
function AppProviders({ children }: { children: React.ReactNode }) {
  return (
    <AuthProvider>
      <ThemeProvider>
        <NotificationProvider>
          <CartProvider>
            {children}
          </CartProvider>
        </NotificationProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}

function App() {
  return (
    <AppProviders>
      <Router />
    </AppProviders>
  );
}
```

### Context with Reducer

For complex state transitions, combine Context with `useReducer`:

```tsx
function AuthProvider({ children }) {
  const [state, dispatch] = useReducer(authReducer, {
    user: null,
    token: null,
    status: 'idle',
  });

  const login = useCallback(async (credentials) => {
    dispatch({ type: 'LOGIN_START' });
    try {
      const data = await api.login(credentials);
      dispatch({ type: 'LOGIN_SUCCESS', payload: data });
    } catch (error) {
      dispatch({ type: 'LOGIN_ERROR', payload: error });
    }
  }, []);

  return (
    <AuthContext value={{ ...state, login }}>
      {children}
    </AuthContext>
  );
}
```

## Common Pitfalls

1. **Putting everything in one giant context** — A single context with user data, theme, cart, and notifications means every consumer re-renders when any value changes. Split into focused contexts.
2. **Creating a new object on every render** — The `value` prop of a provider should be memoized if the provider's parent re-renders frequently: `const value = useMemo(() => ({ theme, toggle }), [theme])`.

## Best Practices

1. **One context per domain** — Keep auth, theme, notifications, and cart in separate contexts. This follows the single responsibility principle and minimizes unnecessary re-renders.
2. **Memoize the provider value** — If the provider component can re-render due to external causes, wrap the context value in `useMemo` to prevent cascading re-renders.

## Summary

- Encapsulate context logic in dedicated provider components with custom consumer hooks.
- Split state and dispatch into separate contexts when many components only need one or the other.
- Compose multiple providers in an `AppProviders` wrapper to keep the app root clean.

## Code Examples

**Split context pattern separating read (state) from write (dispatch)**

```tsx
// Split state and dispatch contexts for performance
const TodosContext = createContext<Todo[]>([]);
const TodosDispatchContext = createContext<React.Dispatch<TodoAction> | null>(null);

function TodosProvider({ children }: { children: React.ReactNode }) {
  const [todos, dispatch] = useReducer(todosReducer, []);

  return (
    <TodosContext value={todos}>
      <TodosDispatchContext value={dispatch}>
        {children}
      </TodosDispatchContext>
    </TodosContext>
  );
}

function useTodos() { return useContext(TodosContext); }
function useTodosDispatch() {
  const dispatch = useContext(TodosDispatchContext);
  if (!dispatch) throw new Error('Must be inside TodosProvider');
  return dispatch;
}
```


## Resources

- [Scaling Up with Reducer and Context](https://react.dev/learn/scaling-up-with-reducer-and-context) — Official guide on combining useReducer with Context

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*