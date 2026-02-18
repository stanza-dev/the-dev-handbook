---
source_course: "react-server-components"
source_lesson: "react-server-components-async-components"
---

# Async Server Components: Direct Data Fetching

Server Components can be `async` functions, allowing you to `await` data directly in your component.

## The Simplicity of Async Components

```tsx
// No useEffect, no loading state management, no API routes
async function UserProfile({ userId }) {
  const user = await db.users.findUnique({ where: { id: userId } });
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## Compare: Before and After RSC

### Before (Client Component)

```tsx
'use client';

function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then(res => res.json())
      .then(data => {
        setUser(data);
        setLoading(false);
      })
      .catch(err => {
        setError(err);
        setLoading(false);
      });
  }, [userId]);
  
  if (loading) return <Spinner />;
  if (error) return <Error message={error.message} />;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

### After (Server Component)

```tsx
async function UserProfile({ userId }) {
  const user = await db.users.findUnique({ where: { id: userId } });
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}
```

## Parallel Data Fetching

Fetch multiple resources in parallel:

```tsx
async function Dashboard({ userId }) {
  // Start all fetches simultaneously
  const [user, posts, notifications] = await Promise.all([
    getUser(userId),
    getUserPosts(userId),
    getNotifications(userId)
  ]);
  
  return (
    <div>
      <UserHeader user={user} />
      <PostList posts={posts} />
      <NotificationBell notifications={notifications} />
    </div>
  );
}
```

## Sequential vs Parallel

```tsx
// ❌ Sequential - slower
async function Page() {
  const user = await getUser();       // Wait...
  const posts = await getPosts();     // Then wait...
  const comments = await getComments(); // Then wait...
  // Total: sum of all fetch times
}

// ✅ Parallel - faster
async function Page() {
  const [user, posts, comments] = await Promise.all([
    getUser(),
    getPosts(),
    getComments()
  ]);
  // Total: longest fetch time only
}
```

## Streaming with Suspense

Wrap async components in Suspense for progressive loading:

```tsx
async function Page() {
  return (
    <div>
      <Header /> {/* Renders immediately */}
      
      <Suspense fallback={<UserSkeleton />}>
        <UserProfile /> {/* Streams when ready */}
      </Suspense>
      
      <Suspense fallback={<PostsSkeleton />}>
        <RecentPosts /> {/* Streams when ready */}
      </Suspense>
    </div>
  );
}

async function UserProfile() {
  const user = await getUser(); // Takes 200ms
  return <Profile user={user} />;
}

async function RecentPosts() {
  const posts = await getPosts(); // Takes 500ms
  return <PostList posts={posts} />;
}
```

Users see:
1. Header immediately
2. UserProfile after 200ms
3. RecentPosts after 500ms

## Resources

- [Async Components](https://react.dev/reference/rsc/server-components#async-components-with-server-components) — Official React documentation on async Server Components

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*