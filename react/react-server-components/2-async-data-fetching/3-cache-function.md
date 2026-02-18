---
source_course: "react-server-components"
source_lesson: "react-server-components-cache-function"
---

# cache(): Request-Level Memoization

The `cache` function from React lets you memoize expensive data fetches or computations within a single server request.

## The Problem

Without caching, the same data might be fetched multiple times:

```tsx
// Both components fetch the same user!
async function UserHeader({ userId }) {
  const user = await getUser(userId); // Fetch #1
  return <h1>{user.name}</h1>;
}

async function UserSidebar({ userId }) {
  const user = await getUser(userId); // Fetch #2 - duplicate!
  return <aside>{user.bio}</aside>;
}
```

## The Solution: cache()

```tsx
import { cache } from 'react';

// Wrap your data fetching function
const getUser = cache(async (userId) => {
  console.log('Fetching user:', userId);
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});

// Now both components share the same fetch
async function UserHeader({ userId }) {
  const user = await getUser(userId); // Fetches
  return <h1>{user.name}</h1>;
}

async function UserSidebar({ userId }) {
  const user = await getUser(userId); // Uses cached result!
  return <aside>{user.bio}</aside>;
}
```

## How cache() Works

1. **First call**: Executes the function, stores the result
2. **Subsequent calls** (same arguments): Returns cached result
3. **Request ends**: Cache is cleared

```tsx
const getUser = cache(async (id) => {
  console.log('Fetching:', id);
  return await db.users.findUnique({ where: { id } });
});

// During a single request:
await getUser(1); // Logs: "Fetching: 1"
await getUser(1); // No log - cached!
await getUser(2); // Logs: "Fetching: 2"
await getUser(1); // No log - still cached!
```

## Practical Example: Shared Data

```tsx
import { cache } from 'react';
import { cookies } from 'next/headers';

// Cache the current user for the entire request
export const getCurrentUser = cache(async () => {
  const token = cookies().get('auth-token')?.value;
  if (!token) return null;
  
  const user = await verifyToken(token);
  return user;
});

// Use anywhere in your Server Components
async function Header() {
  const user = await getCurrentUser();
  return <nav>{user ? `Hi, ${user.name}` : 'Sign in'}</nav>;
}

async function Sidebar() {
  const user = await getCurrentUser(); // Same user, no extra DB call
  return <aside>{user?.preferences}</aside>;
}

async function MainContent() {
  const user = await getCurrentUser(); // Still cached!
  return <main>Welcome back, {user?.name}</main>;
}
```

## cache() vs Other Caching

| Method | Scope | Duration |
|--------|-------|----------|
| `cache()` | Single request | Request lifetime |
| `fetch` cache | Across requests | Configurable |
| `unstable_cache` | Across requests | Configurable |
| External cache (Redis) | Global | Configurable |

## Important Notes

1. **Request-scoped**: Cache is cleared after each request
2. **Argument-based**: Different arguments = different cache entries
3. **Server only**: Only works in Server Components

```tsx
// Arguments matter!
const getData = cache(async (id, options) => { ... });

await getData(1, { full: true });  // Cache miss
await getData(1, { full: true });  // Cache hit
await getData(1, { full: false }); // Cache miss - different options!
```

## Resources

- [cache API Reference](https://react.dev/reference/react/cache) — Official React documentation for the cache function

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*