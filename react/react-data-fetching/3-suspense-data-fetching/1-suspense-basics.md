---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-suspense-basics"
---

# Understanding Suspense

Suspense lets you declaratively specify loading states for parts of your component tree.

## The Problem Suspense Solves

Traditional approach - imperative loading states:

```jsx
function Profile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  // You manage loading state manually
  if (loading) return <Spinner />;
  return <ProfileContent user={user} />;
}
```

With Suspense - declarative loading states:

```jsx
function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <Profile />
    </Suspense>
  );
}

function Profile() {
  const user = use(fetchUser()); // Suspends while loading
  return <ProfileContent user={user} />;
}
```

## How Suspense Works

1. Component "suspends" (throws a promise)
2. React catches it and shows the fallback
3. When promise resolves, React re-renders
4. Component receives the data

## Basic Usage

```jsx
import { Suspense } from 'react';

function App() {
  return (
    <div>
      <h1>My App</h1>
      <Suspense fallback={<div>Loading profile...</div>}>
        <ProfileDetails />
      </Suspense>
    </div>
  );
}
```

## Nested Suspense Boundaries

```jsx
function App() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Header />
      <Suspense fallback={<MainSkeleton />}>
        <MainContent />
      </Suspense>
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
    </Suspense>
  );
}
```

Each Suspense boundary can show its own fallback independently.

## Coordinating Multiple Components

Wrap related components in one boundary to reveal together:

```jsx
function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        {/* Both load together */}
        <Biography artistId={artist.id} />
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}
```

📚 **Learn more**: [Suspense Reference](https://react.dev/reference/react/Suspense)

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*