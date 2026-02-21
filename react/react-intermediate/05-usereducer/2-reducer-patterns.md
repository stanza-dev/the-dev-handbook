---
source_course: "react-intermediate"
source_lesson: "react-reducer-patterns"
---

# Reducer Patterns

## Introduction

As you build more complex features with `useReducer`, you will encounter common patterns for structuring reducers, handling derived state, managing nested objects, and modeling state machines. This lesson covers the patterns that make reducers scalable and maintainable in real applications.

## Key Concepts

- **Action Creators**: Functions that create action objects, centralizing the logic for building actions with payloads.
- **Immer Integration**: Using Immer to write "mutative" reducer code that produces immutable results behind the scenes.
- **State Machines**: Modeling state as a finite set of states with defined transitions, preventing impossible states.
- **Derived State**: Computing values from reducer state rather than storing redundant data.

## Real World Context

A file upload feature has states like idle, selecting, uploading (with progress), success, and error. Each state allows only certain transitions: you cannot go from idle to uploading without first selecting a file. Modeling this as a state machine in a reducer makes impossible states unrepresentable and eliminates entire classes of bugs.

## Deep Dive

### Action Creators

Instead of building action objects inline, create helper functions:

```tsx
// Action creators — centralize action construction
const actions = {
  addItem: (product: Product) => ({
    type: 'ADD_ITEM' as const,
    payload: product,
  }),
  removeItem: (id: string) => ({
    type: 'REMOVE_ITEM' as const,
    payload: id,
  }),
  updateQuantity: (id: string, qty: number) => ({
    type: 'UPDATE_QUANTITY' as const,
    payload: { id, qty },
  }),
  clearCart: () => ({ type: 'CLEAR_CART' as const }),
};

// Usage is more readable
dispatch(actions.addItem(product));
dispatch(actions.updateQuantity('abc', 3));
```

### State Machine Pattern

Model valid states and transitions explicitly:

```tsx
type UploadState =
  | { status: 'idle' }
  | { status: 'selected'; file: File }
  | { status: 'uploading'; file: File; progress: number }
  | { status: 'success'; url: string }
  | { status: 'error'; error: string };

type UploadAction =
  | { type: 'SELECT_FILE'; payload: File }
  | { type: 'START_UPLOAD' }
  | { type: 'PROGRESS'; payload: number }
  | { type: 'UPLOAD_SUCCESS'; payload: string }
  | { type: 'UPLOAD_ERROR'; payload: string }
  | { type: 'RESET' };

function uploadReducer(state: UploadState, action: UploadAction): UploadState {
  switch (action.type) {
    case 'SELECT_FILE':
      return { status: 'selected', file: action.payload };
    case 'START_UPLOAD':
      if (state.status !== 'selected') return state; // Guard: can only start from selected
      return { status: 'uploading', file: state.file, progress: 0 };
    case 'PROGRESS':
      if (state.status !== 'uploading') return state;
      return { ...state, progress: action.payload };
    case 'UPLOAD_SUCCESS':
      return { status: 'success', url: action.payload };
    case 'UPLOAD_ERROR':
      return { status: 'error', error: action.payload };
    case 'RESET':
      return { status: 'idle' };
    default:
      return state;
  }
}
```

### Derived State from Reducer

Never store values that can be computed from existing state:

```tsx
type CartState = {
  items: Array<{ id: string; name: string; price: number; quantity: number }>;
  // Do NOT store: total, itemCount — derive them
};

function CartSummary() {
  const [cart, dispatch] = useReducer(cartReducer, { items: [] });

  // Derived values computed during render
  const itemCount = cart.items.reduce((sum, item) => sum + item.quantity, 0);
  const total = cart.items.reduce((sum, item) => sum + item.price * item.quantity, 0);

  return <p>{itemCount} items - Total: ${total.toFixed(2)}</p>;
}
```

### Lazy Initialization

For expensive initial state computation, pass an init function:

```tsx
function init(savedData: string | null): CartState {
  if (savedData) {
    try { return JSON.parse(savedData); }
    catch { /* fall through */ }
  }
  return { items: [] };
}

const [cart, dispatch] = useReducer(
  cartReducer,
  localStorage.getItem('cart'),  // raw initialArg
  init                             // init function transforms it
);
```

## Common Pitfalls

1. **Storing derived state in the reducer** — Storing `total` alongside `items` means you must update both on every item change. Compute `total` from `items` during render instead.
2. **Missing guards for invalid transitions** — A reducer should handle (or ignore) actions that are not valid in the current state. Without guards, you can reach impossible states.

## Best Practices

1. **Use discriminated union types** — TypeScript's discriminated unions for both state and actions give you exhaustive checking and prevent impossible state combinations.
2. **Keep the reducer pure and move side effects outside** — The reducer computes the next state; the component triggers side effects (API calls, navigation) based on the new state.

## Summary

- Action creators make dispatch calls more readable and centralize action construction.
- State machines model valid states and transitions explicitly, preventing impossible states.
- Derive values from state during render instead of storing redundant data in the reducer.

## Code Examples

**Form state machine with guards preventing invalid transitions**

```tsx
type FormState =
  | { status: 'editing'; values: Record<string, string>; errors: Record<string, string> }
  | { status: 'submitting'; values: Record<string, string> }
  | { status: 'submitted' }
  | { status: 'error'; values: Record<string, string>; serverError: string };

type FormAction =
  | { type: 'UPDATE_FIELD'; field: string; value: string }
  | { type: 'SUBMIT' }
  | { type: 'SUBMIT_SUCCESS' }
  | { type: 'SUBMIT_ERROR'; error: string };

function formReducer(state: FormState, action: FormAction): FormState {
  switch (action.type) {
    case 'UPDATE_FIELD':
      if (state.status !== 'editing') return state;
      return { ...state, values: { ...state.values, [action.field]: action.value } };
    case 'SUBMIT':
      if (state.status !== 'editing') return state;
      return { status: 'submitting', values: state.values };
    case 'SUBMIT_SUCCESS':
      return { status: 'submitted' };
    case 'SUBMIT_ERROR':
      if (state.status !== 'submitting') return state;
      return { status: 'error', values: state.values, serverError: action.error };
    default:
      return state;
  }
}
```


## Resources

- [Extracting State Logic into a Reducer](https://react.dev/learn/extracting-state-logic-into-a-reducer) — Official guide on structuring reducers

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*