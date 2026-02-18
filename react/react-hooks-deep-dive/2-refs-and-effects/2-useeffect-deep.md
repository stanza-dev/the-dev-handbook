---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-useeffect-deep"
---

# useEffect: Synchronizing with External Systems

`useEffect` is React's way of letting you synchronize a component with an external system. It runs after the render is committed to the screen.

## The Mental Model

Think of effects as **synchronization**, not lifecycle events:

```tsx
useEffect(() => {
  // This code synchronizes with `roomId`
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  
  return () => {
    // Cleanup: stop synchronizing
    connection.disconnect();
  };
}, [roomId]); // Re-synchronize when roomId changes
```

## Dependency Array Rules

```tsx
// 🔴 Missing dependency - bug!
useEffect(() => {
  fetchData(userId); // userId is used but not in deps
}, []);

// ✅ Correct - includes all reactive values
useEffect(() => {
  fetchData(userId);
}, [userId]);

// ✅ No dependencies - runs after every render
useEffect(() => {
  console.log('Rendered');
});

// ✅ Empty array - runs only on mount
useEffect(() => {
  console.log('Mounted');
  return () => console.log('Unmounted');
}, []);
```

## Common Patterns

### Data Fetching with Cleanup

```tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    let cancelled = false;

    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        
        if (!cancelled) {
          setUser(data);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err as Error);
        }
      }
    }

    fetchUser();

    return () => {
      cancelled = true; // Prevent state updates after unmount
    };
  }, [userId]);

  // ...
}
```

### Event Listeners

```tsx
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}
```

## What NOT to Use Effects For

❌ **Transforming data for rendering** - Do it during render
❌ **Handling user events** - Use event handlers
❌ **Resetting state when props change** - Use a key instead
❌ **Sharing logic between event handlers** - Extract a function

## Resources

- [useEffect API Reference](https://react.dev/reference/react/useEffect) — Official React documentation for useEffect hook
- [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect) — Learn when effects are unnecessary

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*