---
source_course: "react"
source_lesson: "react-useeffect-basics"
---

# useEffect Hook

`useEffect` lets you perform side effects in function components.

## What are Side Effects?

- Fetching data
- Setting up subscriptions
- Manually changing the DOM
- Timers (setTimeout, setInterval)

## Syntax

```jsx
useEffect(() => {
  // Effect code
  return () => {
    // Cleanup (optional)
  };
}, [dependencies]);
```

## Code Examples

**useEffect with cleanup and dependencies**

```tsx
import { useState, useEffect } from 'react';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    
    // Cleanup function
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]); // Re-run when these change

  return <div>Connected to {roomId}</div>;
}
```


## Resources

- [useEffect Documentation](https://react.dev/reference/react/useEffect) — Official useEffect API reference
- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect) — Learn when NOT to use useEffect

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*