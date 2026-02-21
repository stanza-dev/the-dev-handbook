---
source_course: "react"
source_lesson: "react-usestate-basics"
---

# useState Basics

## Introduction

State is what makes React components interactive. While props are data passed in from a parent, state is data that a component owns and can change over time. The `useState` hook is the primary way to add state to function components. Every time state changes, React re-renders the component to reflect the new data.

## Key Concepts

- **State Variable**: A value that persists across re-renders and whose changes trigger a new render.
- **Setter Function**: The function returned by `useState` that lets you update the state value.
- **Initial Value**: The argument passed to `useState`, used only on the first render.
- **Re-Render**: When state changes, React calls your component function again with the new state value.

## Real World Context

Consider a shopping cart. The number of items, the selected shipping method, and whether the promo code input is visible are all state. Each time the user adds an item or toggles a section, the relevant state changes and React re-renders the affected parts of the UI. Without state, the UI would be static and unresponsive to user actions.

## Deep Dive

### Basic Syntax

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

`useState(0)` returns a pair: the current value (`count`) and a setter (`setCount`). The initial value `0` is used only on the first render.

### Multiple State Variables

You can call `useState` multiple times for independent pieces of state:

```tsx
function SignupForm() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [agreedToTerms, setAgreedToTerms] = useState(false);

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!agreedToTerms) return;
    registerUser(email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="email" value={email} onChange={e => setEmail(e.target.value)} />
      <input type="password" value={password} onChange={e => setPassword(e.target.value)} />
      <label>
        <input type="checkbox" checked={agreedToTerms} onChange={e => setAgreedToTerms(e.target.checked)} />
        I agree to the terms
      </label>
      <button type="submit">Sign Up</button>
    </form>
  );
}
```

### Lazy Initialization

If computing the initial value is expensive, pass a function to avoid recalculating on every render:

```tsx
// Expensive: runs JSON.parse on every render (result discarded after first)
const [data, setData] = useState(JSON.parse(localStorage.getItem('data') || '{}'));

// Lazy: runs JSON.parse only on the first render
const [data, setData] = useState(() => JSON.parse(localStorage.getItem('data') || '{}'));
```

### Rules of Hooks

`useState` (and all hooks) must follow two rules:
1. **Call at the top level** — never inside loops, conditions, or nested functions.
2. **Call only in React functions** — component functions or custom hooks.

## Common Pitfalls

1. **Expecting state to update immediately** — State updates are asynchronous. Logging `count` right after `setCount(count + 1)` will show the old value. The new value is available on the next render.
2. **Using state for derived data** — If a value can be computed from props or other state, do not store it in state. Compute it during render instead: `const fullName = firstName + ' ' + lastName;`.

## Best Practices

1. **Keep state minimal** — Only store the source of truth. Derive everything else during render. This eliminates bugs caused by state getting out of sync.
2. **Use multiple useState calls for unrelated values** — Grouping unrelated values into a single object makes updates verbose. Keep independent values in separate state variables.

## Summary

- `useState` returns a `[value, setter]` pair that persists across renders and triggers re-renders on change.
- Pass a function to `useState` for expensive initial value computations (lazy initialization).
- Always call hooks at the top level of your component, never inside conditions or loops.

## Code Examples

**useState with a derived value computed during render**

```tsx
import { useState } from 'react';

function TemperatureConverter() {
  const [celsius, setCelsius] = useState(0);
  // Derived value — no need for separate state
  const fahrenheit = (celsius * 9) / 5 + 32;

  return (
    <div>
      <label>
        Celsius:
        <input
          type="number"
          value={celsius}
          onChange={e => setCelsius(Number(e.target.value))}
        />
      </label>
      <p>{celsius}°C = {fahrenheit.toFixed(1)}°F</p>
    </div>
  );
}
```


## Resources

- [useState Documentation](https://react.dev/reference/react/useState) — Official useState API reference
- [State: A Component's Memory](https://react.dev/learn/state-a-components-memory) — Conceptual guide to state in React

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*