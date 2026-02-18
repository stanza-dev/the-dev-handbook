---
source_course: "react"
source_lesson: "react-usereducer-basics"
---

# useReducer Hook

`useReducer` is an alternative to `useState` for complex state logic.

## When to Use

- State has multiple sub-values
- Next state depends on previous state
- Complex update logic

## Syntax

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- `reducer`: Function `(state, action) => newState`
- `dispatch`: Function to send actions

## Code Examples

**Counter with useReducer**

```tsx
import { useReducer } from 'react';

type State = { count: number };
type Action = 
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'reset' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*