---
source_course: "react"
source_lesson: "react-suspense-data-fetching"
---

# Suspense for Data Fetching

## Introduction

While Suspense traditionally worked with lazy loading, React 19's `use` API and Suspense-enabled libraries let you use it for data fetching too.

## Key Concepts

**Suspense for data** works the same as for code - components that need data suspend until it's available, showing a fallback in the meantime.

## Real World Context

In modern React apps:
- TanStack Query, SWR, and Relay support Suspense mode
- Server Components can async/await directly
- The `use` API reads promises in client components
- Streaming SSR sends HTML as data becomes available

## Deep Dive

### Using the `use` API

React 19's `use` reads promises in render:

```tsx
import { use, Suspense } from 'react';

function UserProfile({ userPromise }) {
  const user = use(userPromise); // Suspends until resolved
  return <h1>{user.name}</h1>;
}

// Parent creates the promise (not in render!)
function ProfilePage({ userId }) {
  const userPromise = fetchUser(userId); // Cached or memoized
  
  return (
    <Suspense fallback={<ProfileSkeleton />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

### With TanStack Query

```tsx
import { useSuspenseQuery } from '@tanstack/react-query';

function Posts() {
  const { data: posts } = useSuspenseQuery({
    queryKey: ['posts'],
    queryFn: fetchPosts
  });

  return posts.map(post => (
    <article key={post.id}>{post.title}</article>
  ));
}

function PostsPage() {
  return (
    <Suspense fallback={<PostsSkeleton />}>
      <Posts />
    </Suspense>
  );
}
```

### Parallel vs Sequential Loading

**Sequential (Waterfall):**
```tsx
<Suspense fallback={<UserSkeleton />}>
  <User id={userId} /> {/* Loads first */}
  <Suspense fallback={<PostsSkeleton />}>
    <Posts userId={userId} /> {/* Waits for User */}
  </Suspense>
</Suspense>
```

**Parallel:**
```tsx
<Suspense fallback={<PageSkeleton />}>
  <User id={userId} /> {/* Both load simultaneously */}
  <Posts userId={userId} />
</Suspense>
```

### Server Components + Suspense

In frameworks like Next.js, async Server Components automatically integrate with Suspense:

```tsx
// Server Component - can await directly
async function PostList() {
  const posts = await db.posts.findMany();
  return posts.map(post => <Post key={post.id} {...post} />);
}

// Page with streaming
export default function Page() {
  return (
    <div>
      <Header /> {/* Sent immediately */}
      <Suspense fallback={<PostsSkeleton />}>
        <PostList /> {/* Streamed when ready */}
      </Suspense>
    </div>
  );
}
```

## Common Pitfalls

1. **Creating promises in render**: This causes infinite suspends. Create/cache promises outside the component or use a data library.
2. **Missing error handling**: Suspended promises that reject need an error boundary to catch them.
3. **Suspense waterfalls**: Nested Suspense can cause sequential loading. Lift boundaries up to enable parallel fetching.

## Best Practices

- Use data fetching libraries with Suspense support (TanStack Query, SWR)
- Pair every Suspense boundary with an ErrorBoundary
- Start fetching data as early as possible (route loaders, server components)
- Cache or dedupe promises to avoid duplicate fetches
- Consider loading priority - what should the user see first?
- Use skeleton screens that match your actual UI layout

## Summary

Suspense for data fetching lets you declaratively handle loading states for async data. Use the `use` API or Suspense-enabled libraries. Remember to handle errors and avoid creating waterfalls.

## Code Examples

**Suspense data fetching with caching and error handling**

```tsx
import { Suspense, use } from 'react';
import { ErrorBoundary } from 'react-error-boundary';

// Cache for promises to avoid re-fetching
const cache = new Map<string, Promise<any>>();

function fetchWithCache<T>(key: string, fetcher: () => Promise<T>): Promise<T> {
  if (!cache.has(key)) {
    cache.set(key, fetcher());
  }
  return cache.get(key)!;
}

// Component that suspends
function UserData({ userId }: { userId: string }) {
  const userPromise = fetchWithCache(
    `user-${userId}`,
    () => fetch(`/api/users/${userId}`).then(r => r.json())
  );
  
  const user = use(userPromise);
  
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Error fallback component
function ErrorFallback({ error, resetErrorBoundary }) {
  return (
    <div className="error">
      <h2>Something went wrong</h2>
      <p>{error.message}</p>
      <button onClick={resetErrorBoundary}>Try again</button>
    </div>
  );
}

// Combined Suspense + Error handling
function UserProfile({ userId }: { userId: string }) {
  return (
    <ErrorBoundary
      FallbackComponent={ErrorFallback}
      onReset={() => cache.delete(`user-${userId}`)}
    >
      <Suspense fallback={<UserSkeleton />}>
        <UserData userId={userId} />
      </Suspense>
    </ErrorBoundary>
  );
}
```


## Resources

- [use Reference](https://react.dev/reference/react/use) — Official documentation for the use API
- [TanStack Query Suspense](https://tanstack.com/query/latest/docs/framework/react/guides/suspense) — Using Suspense with TanStack Query

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*