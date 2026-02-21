---
source_course: "react-intermediate"
source_lesson: "react-fetch-useeffect"
---

# Data Fetching with useEffect

## Introduction

The Fetch API is the standard browser interface for making HTTP requests. Combined with React's useEffect hook, it provides a straightforward way to load data into your components. Understanding this fundamental pattern is essential before moving to higher-level data fetching libraries.

## Key Concepts

- **Fetch API**: A promise-based browser API for making network requests, replacing the older XMLHttpRequest.
- **useEffect for data fetching**: Using the effect hook to trigger fetches after render, keeping the rendering synchronous.
- **AbortController**: A browser API for cancelling fetch requests, essential for cleanup when components unmount or dependencies change.

## Real World Context

A user profile page needs to fetch user data when the component mounts and re-fetch when the user ID in the URL changes. Without proper cleanup, navigating quickly between profiles causes race conditions where stale data from a slow earlier request overwrites fresh data from a fast later request.

## Deep Dive

The basic fetch-in-useEffect pattern involves three pieces of state: the data itself, a loading flag, and an error. The AbortController is created inside the effect and its abort method is called in the cleanup function.

\`\`\`tsx
import { useState, useEffect } from 'react';

type User = { id: number; name: string; email: string };

function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();

    async function fetchUser() {
      try {
        setLoading(true);
        setError(null);
        const res = await fetch(\`/api/users/\${userId}\`, {
          signal: controller.signal,
        });
        if (!res.ok) throw new Error('Failed to fetch');
        const data = await res.json();
        setUser(data);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    }

    fetchUser();
    return () => controller.abort();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return <h1>{user.name}</h1>;
}
\`\`\`

The AbortController is critical for two reasons. First, it prevents memory leaks by cancelling requests when the component unmounts. Second, it prevents race conditions when the userId dependency changes rapidly. Each new effect instance aborts the previous request before starting a new one.

## Common Pitfalls

1. **Forgetting cleanup** — Without AbortController, unmounted components attempt to update state, causing the React "Can't perform a state update on an unmounted component" warning.
2. **Not handling the AbortError** — When you abort a fetch, it throws an AbortError. You must filter this out in your catch block to avoid showing false error messages.

## Best Practices

1. **Always use AbortController** — Include cleanup in every fetch effect to prevent race conditions and memory leaks.
2. **Set loading state before fetch** — Reset loading and error at the start of each fetch to ensure a clean state for the new request.

## Summary

- The Fetch API with useEffect is the fundamental data fetching pattern in React.
- AbortController prevents race conditions and memory leaks through effect cleanup.
- Always handle loading, error, and success states for a good user experience.

## Code Examples

**Fetch with loading/error states and AbortController cleanup**

```tsx
import { useState, useEffect } from 'react';

type User = { id: number; name: string; email: string };

function UserProfile({ userId }: { userId: number }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  useEffect(() => {
    const controller = new AbortController();
    async function fetchUser() {
      try {
        setLoading(true);
        setError(null);
        const res = await fetch(`/api/users/${userId}`, {
          signal: controller.signal,
        });
        if (!res.ok) throw new Error('Failed to fetch');
        setUser(await res.json());
      } catch (err) {
        if (err.name !== 'AbortError') setError(err.message);
      } finally {
        setLoading(false);
      }
    }
    fetchUser();
    return () => controller.abort();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <h1>{user?.name}</h1>;
}
```


## Resources

- [Fetching data with Effects](https://react.dev/reference/react/useEffect#fetching-data-with-effects) — Official React guide on data fetching in effects

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*