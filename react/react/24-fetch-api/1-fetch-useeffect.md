---
source_course: "react"
source_lesson: "react-fetch-useeffect"
---

# Data Fetching with Fetch API

The Fetch API provides a simple way to make HTTP requests.

## Basic Pattern

1. Use `useEffect` for the fetch call
2. Store data in state
3. Handle loading and error states
4. Clean up with AbortController

## Code Examples

**Fetch with loading/error states and cleanup**

```tsx
import { useState, useEffect } from 'react';

type User = {
  id: number;
  name: string;
  email: string;
};

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
        
        const response = await fetch(
          `/api/users/${userId}`,
          { signal: controller.signal }
        );
        
        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }
        
        const data = await response.json();
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
    
    // Cleanup: abort fetch on unmount or userId change
    return () => controller.abort();
  }, [userId]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```


---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*