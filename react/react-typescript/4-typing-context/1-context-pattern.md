---
source_course: "react-typescript"
source_lesson: "react-typescript-typed-context-pattern"
---

# Typed Context Pattern

Creating type-safe context with a clean API.

## The Problem with Default Values

```tsx
// ❌ Problem: null default doesn't match User type
const UserContext = createContext<User>(null);

// ❌ Problem: Empty object isn't a valid User
const UserContext = createContext<User>({} as User);
```

## Solution: Allow null, Create Custom Hook

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// Define types
type User = {
  id: string;
  name: string;
  email: string;
};

type AuthContextType = {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
};

// Create context with null default
const AuthContext = createContext<AuthContextType | null>(null);

// Custom hook with null check
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Provider component
type AuthProviderProps = {
  children: ReactNode;
};

function AuthProvider({ children }: AuthProviderProps) {
  const [user, setUser] = useState<User | null>(null);

  const login = async (email: string, password: string) => {
    const user = await authApi.login(email, password);
    setUser(user);
  };

  const logout = () => {
    setUser(null);
  };

  const value: AuthContextType = {
    user,
    login,
    logout,
    isAuthenticated: !!user
  };

  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

export { AuthProvider, useAuth };
```

## Usage

```tsx
// App.tsx
function App() {
  return (
    <AuthProvider>
      <Dashboard />
    </AuthProvider>
  );
}

// Dashboard.tsx
function Dashboard() {
  const { user, logout, isAuthenticated } = useAuth();
  // All typed correctly!
  
  if (!isAuthenticated) {
    return <LoginForm />;
  }

  return (
    <div>
      <h1>Welcome, {user?.name}</h1>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

## Benefits

1. **Type safety** - All context values properly typed
2. **Runtime safety** - Error if used outside provider
3. **Clean API** - No need to check for undefined
4. **Autocomplete** - Full IntelliSense support

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*