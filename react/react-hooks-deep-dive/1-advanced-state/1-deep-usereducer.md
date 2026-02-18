---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-deep-usereducer"
---

# useReducer: The Power of Predictable State

`useReducer` is React's answer to complex state management. While `useState` is perfect for simple values, `useReducer` shines when you have:

- **Multiple sub-values** that need to update together
- **Complex state logic** where the next state depends on the previous one
- **Related state transitions** that should be handled consistently

## The Reducer Pattern

A reducer is a pure function that takes the current state and an action, then returns a new state:

```tsx
type State = {
  count: number;
  step: number;
  history: number[];
};

type Action =
  | { type: 'increment' }
  | { type: 'decrement' }
  | { type: 'setStep'; payload: number }
  | { type: 'reset' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return {
        ...state,
        count: state.count + state.step,
        history: [...state.history, state.count + state.step]
      };
    case 'decrement':
      return {
        ...state,
        count: state.count - state.step,
        history: [...state.history, state.count - state.step]
      };
    case 'setStep':
      return { ...state, step: action.payload };
    case 'reset':
      return { count: 0, step: 1, history: [] };
    default:
      return state;
  }
}
```

## Using the Hook

```tsx
function Counter() {
  const [state, dispatch] = useReducer(reducer, {
    count: 0,
    step: 1,
    history: []
  });

  return (
    <div>
      <p>Count: {state.count}</p>
      <p>Step: {state.step}</p>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <input
        type="number"
        value={state.step}
        onChange={(e) =>
          dispatch({ type: 'setStep', payload: Number(e.target.value) })
        }
      />
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}
```

## Why useReducer Over useState?

1. **Testability**: Reducers are pure functions—easy to test in isolation
2. **Predictability**: All state transitions are explicit and traceable
3. **Performance**: Pass `dispatch` to children without worrying about reference changes
4. **DevTools**: Works great with debugging tools that can replay actions

## Resources

- [useReducer API Reference](https://react.dev/reference/react/useReducer) — Official React documentation for useReducer hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*