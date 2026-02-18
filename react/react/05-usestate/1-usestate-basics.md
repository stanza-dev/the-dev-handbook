---
source_course: "react"
source_lesson: "react-usestate-basics"
---

# useState Hook

`useState` lets you add state variables to functional components.

## Syntax

```jsx
const [state, setState] = useState(initialValue);
```

- `state`: Current value
- `setState`: Function to update the value
- `initialValue`: Starting value (only used on first render)

## Rules

1. Call at the top level of your component
2. Don't call inside loops, conditions, or nested functions
3. State updates are asynchronous and batched

## Code Examples

**Basic counter with useState**

```tsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <button onClick={() => setCount(0)}>
        Reset
      </button>
    </div>
  );
}
```


## Resources

- [useState Documentation](https://react.dev/reference/react/useState) — Official useState API reference

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*