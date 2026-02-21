---
source_course: "react"
source_lesson: "react-useeffect-basics"
---

# useEffect Basics

## Introduction

React components are meant to be pure functions of their props and state. But real applications need to interact with the outside world: fetching data, setting up subscriptions, updating the document title, or connecting to WebSockets. The `useEffect` hook is React's escape hatch for performing these side effects after the component renders.

## Key Concepts

- **Side Effect**: Any operation that reaches outside the component's render scope — network requests, DOM manipulation, timers, subscriptions.
- **Effect Function**: The callback passed to `useEffect`, which runs after React commits the render to the DOM.
- **Cleanup Function**: An optional function returned from the effect that runs before the effect re-runs or when the component unmounts.
- **Dependency Array**: An array of values that determine when the effect should re-run.

## Real World Context

A chat application needs to open a WebSocket connection when the user enters a room and close it when they leave. A dashboard updates the browser tab title with the current notification count. A search component debounces API calls as the user types. All of these are side effects that `useEffect` manages.

## Deep Dive

### Basic Syntax

```tsx
import { useState, useEffect } from 'react';

useEffect(() => {
  // Effect code runs after render
  return () => {
    // Cleanup code (optional)
  };
}, [/* dependencies */]);
```

### Example: Updating Document Title

```tsx
function NotificationCounter({ count }: { count: number }) {
  useEffect(() => {
    document.title = count > 0 ? `(${count}) Messages` : 'Messages';
  }, [count]);

  return <span>You have {count} unread messages</span>;
}
```

The effect runs after the initial render and again whenever `count` changes.

### Example: Fetching Data

```tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      setLoading(true);
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      if (!cancelled) {
        setUser(data);
        setLoading(false);
      }
    }

    fetchUser();

    return () => {
      cancelled = true; // Prevent setting state on unmounted component
    };
  }, [userId]);

  if (loading) return <Spinner />;
  return <div>{user?.name}</div>;
}
```

The `cancelled` flag in the cleanup prevents setting state after the component unmounts or when the userId changes before the previous fetch completes.

### Timing

Effects run asynchronously after the browser paints the screen. This means:
1. React renders the component (calls the function).
2. React updates the DOM.
3. The browser paints.
4. React runs the effect.

This ensures effects do not block the visual update.

## Common Pitfalls

1. **Using useEffect for derived state** — If you are syncing state with props (`useEffect(() => setX(props.y), [props.y])`), you probably do not need an effect. Compute the value during render instead.
2. **Missing the dependency array** — Omitting the array entirely causes the effect to run after every render, which can create infinite loops if the effect updates state.

## Best Practices

1. **Always include a dependency array** — Even if it is empty. This makes your intent explicit: run once (`[]`), run when dependencies change (`[a, b]`), or run every render (no array, which is rare).
2. **Use the cleanup pattern for subscriptions** — Any effect that sets up a listener, timer, or connection should return a cleanup function that tears it down.

## Summary

- `useEffect` runs side effects after the component renders and the browser paints.
- Return a cleanup function from the effect to unsubscribe, cancel requests, or clear timers.
- Always provide a dependency array to control when the effect re-runs.

## Code Examples

**useEffect with cleanup for a chat room connection**

```tsx
import { useState, useEffect } from 'react';

function ChatRoom({ roomId }: { roomId: string }) {
  const [messages, setMessages] = useState<string[]>([]);

  useEffect(() => {
    const connection = createConnection(roomId);
    connection.on('message', (msg: string) => {
      setMessages(prev => [...prev, msg]);
    });
    connection.connect();

    return () => {
      connection.disconnect();
    };
  }, [roomId]);

  return (
    <div>
      <h2>Room: {roomId}</h2>
      {messages.map((msg, i) => <p key={i}>{msg}</p>)}
    </div>
  );
}
```


## Resources

- [useEffect Documentation](https://react.dev/reference/react/useEffect) — Official useEffect API reference
- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect) — Learn when NOT to use useEffect

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*