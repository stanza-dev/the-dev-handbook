---
source_course: "react"
source_lesson: "react-usecallback"
---

# useCallback Hook

`useCallback` caches a function definition between re-renders.

## When to Use

1. **Passing callbacks to memoized children**: Prevents unnecessary re-renders
2. **Dependencies in useEffect**: Stable function references

## useMemo vs useCallback

```jsx
// These are equivalent:
useCallback(fn, deps)
useMemo(() => fn, deps)
```

`useCallback` caches the function itself, `useMemo` caches the return value.

## Code Examples

**useCallback with memo'd child**

```tsx
import { useCallback, memo } from 'react';

const ExpensiveChild = memo(function ExpensiveChild({ onClick }) {
  console.log('ExpensiveChild rendered');
  return <button onClick={onClick}>Click me</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Without useCallback, this creates a new function every render
  // causing ExpensiveChild to re-render unnecessarily
  const handleClick = useCallback(() => {
    console.log('Clicked!');
  }, []); // Empty deps = never recreated

  return (
    <div>
      <input value={name} onChange={e => setName(e.target.value)} />
      <p>Count: {count}</p>
      {/* ExpensiveChild won't re-render when name changes */}
      <ExpensiveChild onClick={handleClick} />
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*