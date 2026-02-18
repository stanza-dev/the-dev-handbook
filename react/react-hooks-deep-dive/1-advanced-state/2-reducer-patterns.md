---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-reducer-patterns"
---

# Advanced Reducer Patterns

Once you understand the basics of `useReducer`, you can leverage powerful patterns to handle complex scenarios.

## Lazy Initialization

When initial state requires expensive computation, use the third argument:

```tsx
function init(initialCount: number): State {
  // Expensive computation or localStorage read
  const saved = localStorage.getItem('counter');
  return saved
    ? JSON.parse(saved)
    : { count: initialCount, step: 1, history: [] };
}

function Counter({ initialCount = 0 }) {
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  // ...
}
```

The `init` function only runs once during the initial render.

## Immer Integration

For deeply nested state, combine with Immer for immutable updates:

```tsx
import { useImmerReducer } from 'use-immer';

function reducer(draft: State, action: Action) {
  switch (action.type) {
    case 'updateNestedValue':
      // Mutate directly - Immer handles immutability
      draft.user.settings.theme = action.payload;
      break;
    case 'addItem':
      draft.items.push(action.payload);
      break;
  }
}
```

## Action Creators

For type safety and consistency, create action creator functions:

```tsx
const actions = {
  increment: () => ({ type: 'increment' as const }),
  decrement: () => ({ type: 'decrement' as const }),
  setStep: (step: number) => ({ type: 'setStep' as const, payload: step }),
  reset: () => ({ type: 'reset' as const })
};

// Usage
dispatch(actions.setStep(5));
```

## Combining Multiple Reducers

For very complex state, split into multiple reducers:

```tsx
function combineReducers<S>(
  reducers: { [K in keyof S]: (state: S[K], action: Action) => S[K] }
) {
  return (state: S, action: Action): S => {
    const newState = {} as S;
    for (const key in reducers) {
      newState[key] = reducers[key](state[key], action);
    }
    return newState;
  };
}

const rootReducer = combineReducers({
  user: userReducer,
  posts: postsReducer,
  ui: uiReducer
});
```

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*