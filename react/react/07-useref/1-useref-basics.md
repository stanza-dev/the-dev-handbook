---
source_course: "react"
source_lesson: "react-useref-basics"
---

# useRef Hook

`useRef` returns a mutable ref object that persists across renders.

## Two Main Uses

1. **Accessing DOM elements**: Get direct access to a DOM node
2. **Storing mutable values**: Values that don't trigger re-renders

## Key Difference from State

| `useState` | `useRef` |
|------------|----------|
| Changes trigger re-render | Changes don't trigger re-render |
| Returns `[value, setter]` | Returns `{ current: value }` |

## Code Examples

**Using useRef to access DOM element**

```tsx
import { useRef, useEffect } from 'react';

function AutoFocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  useEffect(() => {
    // Focus the input on mount
    inputRef.current?.focus();
  }, []);

  return (
    <input
      ref={inputRef}
      placeholder="I'm focused!"
    />
  );
}
```

**Using useRef to store interval ID**

```tsx
function Stopwatch() {
  const [time, setTime] = useState(0);
  const intervalRef = useRef<number | null>(null);

  const start = () => {
    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
    }
  };

  return (
    <div>
      <p>Time: {time}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </div>
  );
}
```


## Resources

- [useRef Documentation](https://react.dev/reference/react/useRef) — Official useRef API reference

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*