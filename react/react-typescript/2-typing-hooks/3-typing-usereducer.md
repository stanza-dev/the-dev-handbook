---
source_course: "react-typescript"
source_lesson: "react-typescript-typing-usereducer"
---

# Typing useReducer

useReducer benefits greatly from TypeScript's discriminated unions.

## Basic Pattern

```tsx
// State type
type State = {
  count: number;
  error: string | null;
};

// Action types using discriminated union
type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset'; payload: number }
  | { type: 'setError'; payload: string };

// Reducer function
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    case 'decrement':
      return { ...state, count: state.count - 1 };
    case 'reset':
      return { ...state, count: action.payload };
    case 'setError':
      return { ...state, error: action.payload };
    default:
      return state;
  }
}

// Usage
const initialState: State = { count: 0, error: null };

function Counter() {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <span>{state.count}</span>
      <button onClick={() => dispatch({ type: 'increment' })}>
        +
      </button>
      <button onClick={() => dispatch({ type: 'reset', payload: 0 })}>
        Reset
      </button>
    </div>
  );
}
```

## Exhaustive Type Checking

```tsx
function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 };
    case 'decrement':
      return { ...state, count: state.count - 1 };
    // If you forget a case, TypeScript will warn you!
    default:
      // This ensures all cases are handled
      const _exhaustive: never = action;
      return state;
  }
}
```

## Async Actions Pattern

```tsx
type FetchState<T> = 
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

type FetchAction<T> =
  | { type: 'fetch' }
  | { type: 'success'; payload: T }
  | { type: 'error'; payload: string };

function fetchReducer<T>(state: FetchState<T>, action: FetchAction<T>): FetchState<T> {
  switch (action.type) {
    case 'fetch':
      return { status: 'loading' };
    case 'success':
      return { status: 'success', data: action.payload };
    case 'error':
      return { status: 'error', error: action.payload };
  }
}

function useFetch<T>(fetchFn: () => Promise<T>) {
  const [state, dispatch] = useReducer(
    fetchReducer<T>,
    { status: 'idle' }
  );

  const execute = async () => {
    dispatch({ type: 'fetch' });
    try {
      const data = await fetchFn();
      dispatch({ type: 'success', payload: data });
    } catch (e) {
      dispatch({ type: 'error', payload: (e as Error).message });
    }
  };

  return { ...state, execute };
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*