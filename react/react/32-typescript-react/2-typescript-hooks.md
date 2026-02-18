---
source_course: "react"
source_lesson: "react-typescript-hooks"
---

# Typing React Hooks

## Introduction

React hooks are already typed, but you'll often need to provide explicit types for state, refs, and custom hooks.

## Key Concepts

**Generic type parameters** let you specify the types hooks work with: `useState<Type>()`, `useRef<Type>()`, etc.

## Real World Context

Properly typed hooks:
- Prevent runtime type errors in state management
- Enable autocompletion for state and ref values
- Make custom hooks reusable with type safety
- Document expected data shapes

## Deep Dive

### useState

TypeScript infers from the initial value, but explicit types help for complex state:

```tsx
// Inferred as number
const [count, setCount] = useState(0);

// Explicit for objects
type User = {
  id: string;
  name: string;
  email: string;
};

const [user, setUser] = useState<User | null>(null);

// Union types for status
type Status = 'idle' | 'loading' | 'success' | 'error';
const [status, setStatus] = useState<Status>('idle');
```

### useRef

Different types for DOM refs vs mutable values:

```tsx
// DOM element ref - use null initially
const inputRef = useRef<HTMLInputElement>(null);
// Access with: inputRef.current?.focus()

// Mutable value ref - use initial value
const countRef = useRef<number>(0);
// Access with: countRef.current (no null check needed)

// The difference: null initial = read-only .current
// Value initial = mutable .current
```

### useReducer

```tsx
type State = {
  count: number;
  step: number;
};

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number }
  | { type: 'reset' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + state.step };
    case 'decrement':
      return { ...state, count: state.count - state.step };
    case 'setStep':
      return { ...state, step: action.payload };
    case 'reset':
      return { count: 0, step: 1 };
  }
}

const [state, dispatch] = useReducer(reducer, { count: 0, step: 1 });
```

### useContext

```tsx
type AuthContextType = {
  user: User | null;
  login: (credentials: Credentials) => Promise<void>;
  logout: () => void;
};

// Create with undefined default to enforce Provider use
const AuthContext = createContext<AuthContextType | undefined>(undefined);

// Custom hook that throws if used outside Provider
function useAuth(): AuthContextType {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}
```

## Common Pitfalls

1. **Forgetting null for DOM refs**: `useRef<HTMLElement>()` without null causes type errors when accessing `.current`.
2. **Not typing reducer actions**: Untyped actions miss the benefits of discriminated unions.
3. **Using `any` in context**: Loses type safety for all consumers.

## Best Practices

- Let TypeScript infer when possible (simple useState)
- Provide explicit types for null-possible values
- Use discriminated unions for reducer actions
- Create custom hooks that enforce Provider usage for context
- Export types from custom hooks for consumers
- Use `as const` for readonly state values

## Summary

Hooks use generic type parameters for type safety. Provide explicit types for complex state, nullable values, and DOM refs. Use discriminated unions for reducer actions.

## Code Examples

**Typed hooks with useState, useRef, and useReducer**

```tsx
import { useState, useRef, useReducer, useCallback } from 'react';

// Complex state with explicit type
type FormState = {
  values: Record<string, string>;
  errors: Record<string, string>;
  isSubmitting: boolean;
};

function useForm(initialValues: Record<string, string>) {
  const [state, setState] = useState<FormState>({
    values: initialValues,
    errors: {},
    isSubmitting: false
  });

  const setValue = useCallback((name: string, value: string) => {
    setState(prev => ({
      ...prev,
      values: { ...prev.values, [name]: value },
      errors: { ...prev.errors, [name]: '' } // Clear error on change
    }));
  }, []);

  const setError = useCallback((name: string, error: string) => {
    setState(prev => ({
      ...prev,
      errors: { ...prev.errors, [name]: error }
    }));
  }, []);

  return { state, setValue, setError };
}

// DOM ref with proper typing
function SearchInput() {
  const inputRef = useRef<HTMLInputElement>(null);
  
  const focusInput = () => {
    inputRef.current?.focus(); // Safe optional chaining
  };
  
  return (
    <div>
      <input ref={inputRef} type="search" />
      <button onClick={focusInput}>Focus</button>
    </div>
  );
}

// Discriminated union for actions
type TodoAction =
  | { type: 'add'; payload: { text: string } }
  | { type: 'toggle'; payload: { id: string } }
  | { type: 'delete'; payload: { id: string } };

type Todo = { id: string; text: string; done: boolean };

function todosReducer(state: Todo[], action: TodoAction): Todo[] {
  switch (action.type) {
    case 'add':
      return [...state, { 
        id: crypto.randomUUID(), 
        text: action.payload.text, 
        done: false 
      }];
    case 'toggle':
      return state.map(todo =>
        todo.id === action.payload.id
          ? { ...todo, done: !todo.done }
          : todo
      );
    case 'delete':
      return state.filter(todo => todo.id !== action.payload.id);
  }
}
```


## Resources

- [Typing Hooks](https://react.dev/learn/typescript#typing-usestate) — Official guide to typing React hooks

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*