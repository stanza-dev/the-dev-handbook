---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-fetching-with-useeffect"
---

# Fetching with useEffect

`useEffect` is the traditional way to fetch data in React components.

## Basic Pattern

```jsx
import { useState, useEffect } from 'react';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Reset state when userId changes
    setLoading(true);
    setError(null);

    fetch(`/api/users/${userId}`)
      .then(response => {
        if (!response.ok) {
          throw new Error('Failed to fetch');
        }
        return response.json();
      })
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err.message);
        setLoading(false);
      });
  }, [userId]); // Re-fetch when userId changes

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>Hello, {user.name}!</div>;
}
```

## With async/await

You can't make the effect function async directly, but you can define an async function inside:

```jsx
useEffect(() => {
  async function fetchUser() {
    setLoading(true);
    try {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      setUser(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  fetchUser();
}, [userId]);
```

## Handling Race Conditions

Prevent stale responses from overwriting newer data:

```jsx
useEffect(() => {
  let cancelled = false;

  async function fetchUser() {
    setLoading(true);
    try {
      const response = await fetch(`/api/users/${userId}`);
      const data = await response.json();
      
      // Only update if not cancelled
      if (!cancelled) {
        setUser(data);
        setLoading(false);
      }
    } catch (err) {
      if (!cancelled) {
        setError(err.message);
        setLoading(false);
      }
    }
  }

  fetchUser();

  // Cleanup function
  return () => {
    cancelled = true;
  };
}, [userId]);
```

## With AbortController

Actually cancel the request:

```jsx
useEffect(() => {
  const controller = new AbortController();

  async function fetchUser() {
    try {
      const response = await fetch(`/api/users/${userId}`, {
        signal: controller.signal
      });
      const data = await response.json();
      setUser(data);
    } catch (err) {
      if (err.name !== 'AbortError') {
        setError(err.message);
      }
    }
  }

  fetchUser();

  return () => controller.abort();
}, [userId]);
```

📚 **Learn more**: [You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*