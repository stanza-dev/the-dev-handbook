---
source_course: "react-intermediate"
source_lesson: "react-concurrent-rendering-intro"
---

# Understanding Concurrent Rendering

## Introduction

Concurrent rendering is React's ability to prepare multiple versions of your UI simultaneously. This enables React to keep your interface responsive even during expensive state updates.

## Key Concepts

**Concurrent rendering** allows React to interrupt, pause, or abandon a render to prioritize more urgent updates. Unlike traditional synchronous rendering where React must finish what it started, concurrent React can work on multiple state updates at different priority levels.

## Real World Context

Imagine a search interface: the user types in a search box while results filter below. Without concurrent features, each keystroke might cause the entire list to re-render, making the input feel laggy. With concurrent rendering, React prioritizes the input update (urgent) over the list filtering (non-urgent).

## Deep Dive

### How It Works

1. **Interruptible rendering**: React can pause rendering a low-priority update to handle high-priority updates (like user input)
2. **Automatic batching**: Multiple state updates are grouped together for efficiency
3. **Transition API**: Explicitly mark updates as non-urgent with `useTransition` or `startTransition`

### The Mental Model

Think of concurrent React like a smart assistant who can:
- Drop what they're doing when something urgent comes up
- Pick up where they left off when the urgent task is done
- Work on multiple things in the background

```tsx
// React can now pause updating the list
// to immediately respond to the next keystroke
function SearchApp() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  // This update is marked as a transition (low priority)
  const [isPending, startTransition] = useTransition();

  function handleChange(e) {
    // High priority - update input immediately
    setQuery(e.target.value);
    
    // Low priority - can be interrupted
    startTransition(() => {
      setResults(filterResults(e.target.value));
    });
  }

  return (/* ... */);
}
```

## Common Pitfalls

1. **Overusing transitions**: Not every update needs to be a transition. Only use for expensive updates that can afford to be deferred.
2. **Expecting instant updates**: Transitions are intentionally deferred - don't use them for updates that must happen immediately.
3. **Confusing with debouncing**: Transitions don't delay the update, they deprioritize it. The update still happens as soon as React has time.

## Best Practices

- Use transitions for updates that trigger expensive re-renders (large lists, complex calculations)
- Keep urgent interactions (typing, clicking, hovering) outside transitions
- Provide visual feedback with `isPending` to indicate background work
- Combine with `useDeferredValue` for derived values based on deferred state

## Summary

Concurrent rendering lets React work on multiple updates simultaneously, prioritizing urgent user interactions over expensive computations. The `useTransition` hook is the primary API for marking updates as non-urgent transitions.

## Code Examples

**Basic useTransition example for filtering a large list**

```tsx
import { useState, useTransition } from 'react';

function FilterableList({ items }: { items: string[] }) {
  const [query, setQuery] = useState('');
  const [isPending, startTransition] = useTransition();

  // Expensive filtering operation
  const filteredItems = items.filter(item =>
    item.toLowerCase().includes(query.toLowerCase())
  );

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    
    // Wrap the expensive update in a transition
    startTransition(() => {
      setQuery(value);
    });
  }

  return (
    <div>
      <input
        type="text"
        onChange={handleChange}
        placeholder="Search..."
      />
      
      {isPending && <div className="spinner">Updating...</div>}
      
      <ul style={{ opacity: isPending ? 0.7 : 1 }}>
        {filteredItems.map(item => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```


## Resources

- [useTransition Reference](https://react.dev/reference/react/useTransition) — Official React documentation for useTransition
- [Concurrent React Overview](https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react) — Introduction to Concurrent React concepts

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*