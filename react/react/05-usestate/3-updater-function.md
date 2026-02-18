---
source_course: "react"
source_lesson: "react-usestate-updater-function"
---

# Functional Updates

When the new state depends on the previous state, use a function:

```jsx
setCount(prevCount => prevCount + 1);
```

## Why Use It?

1. **Batched updates**: Multiple calls in the same event use the latest value
2. **Closures**: Avoids stale closure issues in async code
3. **Correctness**: Guaranteed to use the most recent state

## Code Examples

**Using functional updates for correct batching**

```tsx
function Counter() {
  const [count, setCount] = useState(0);

  const incrementThrice = () => {
    // ❌ Wrong - all use the same stale `count`
    // setCount(count + 1);
    // setCount(count + 1);
    // setCount(count + 1); // Still only adds 1!

    // ✅ Correct - uses latest value
    setCount(c => c + 1);
    setCount(c => c + 1);
    setCount(c => c + 1); // Adds 3!
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={incrementThrice}>
        +3
      </button>
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*