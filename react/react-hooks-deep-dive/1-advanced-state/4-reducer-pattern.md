---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-context-reducer-pattern"
---

# Combining Context and Reducer

One of the most powerful patterns in React is combining `useContext` with `useReducer` to create a lightweight state management solution.

## The Pattern

```tsx
import { createContext, useContext, useReducer, ReactNode } from 'react';

// Types
type State = {
  user: User | null;
  isLoading: boolean;
  error: string | null;
};

type Action =
  | { type: 'LOGIN_START' }
  | { type: 'LOGIN_SUCCESS'; payload: User }
  | { type: 'LOGIN_ERROR'; payload: string }
  | { type: 'LOGOUT' };

// Reducer
function authReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'LOGIN_START':
      return { ...state, isLoading: true, error: null };
    case 'LOGIN_SUCCESS':
      return { user: action.payload, isLoading: false, error: null };
    case 'LOGIN_ERROR':
      return { ...state, isLoading: false, error: action.payload };
    case 'LOGOUT':
      return { user: null, isLoading: false, error: null };
    default:
      return state;
  }
}

// Contexts - separate for performance
const AuthStateContext = createContext<State | null>(null);
const AuthDispatchContext = createContext<React.Dispatch<Action> | null>(null);

// Provider
function AuthProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(authReducer, {
    user: null,
    isLoading: false,
    error: null
  });

  return (
    <AuthStateContext.Provider value={state}>
      <AuthDispatchContext.Provider value={dispatch}>
        {children}
      </AuthDispatchContext.Provider>
    </AuthStateContext.Provider>
  );
}

// Custom hooks
function useAuthState() {
  const context = useContext(AuthStateContext);
  if (context === null) {
    throw new Error('useAuthState must be used within AuthProvider');
  }
  return context;
}

function useAuthDispatch() {
  const context = useContext(AuthDispatchContext);
  if (context === null) {
    throw new Error('useAuthDispatch must be used within AuthProvider');
  }
  return context;
}

// Convenience hook with actions
function useAuth() {
  const state = useAuthState();
  const dispatch = useAuthDispatch();

  const login = async (credentials: Credentials) => {
    dispatch({ type: 'LOGIN_START' });
    try {
      const user = await authApi.login(credentials);
      dispatch({ type: 'LOGIN_SUCCESS', payload: user });
    } catch (error) {
      dispatch({ type: 'LOGIN_ERROR', payload: error.message });
    }
  };

  const logout = () => dispatch({ type: 'LOGOUT' });

  return { ...state, login, logout };
}
```

## Why Separate Contexts?

Splitting state and dispatch into separate contexts is a performance optimization:

- Components that only dispatch actions won't re-render when state changes
- Components that only read state won't be affected by dispatch reference changes

## Usage

```tsx
function App() {
  return (
    <AuthProvider>
      <Header />
      <Main />
    </AuthProvider>
  );
}

function LoginButton() {
  const { user, isLoading, login, logout } = useAuth();

  if (isLoading) return <Spinner />;
  if (user) return <button onClick={logout}>Logout</button>;
  return <button onClick={() => login(credentials)}>Login</button>;
}
```

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*