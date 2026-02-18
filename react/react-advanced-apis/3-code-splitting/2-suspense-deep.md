---
source_course: "react-advanced-apis"
source_lesson: "react-advanced-apis-suspense-deep"
---

# Suspense: Declarative Loading States

`<Suspense>` lets you display a fallback while waiting for children to load. It works with lazy components, data fetching, and Server Components.

## Basic Usage

```tsx
import { Suspense } from 'react';

function App() {
  return (
    <Suspense fallback={<Spinner />}>
      <SlowComponent />
    </Suspense>
  );
}
```

## Nested Suspense Boundaries

You can nest Suspense boundaries for granular loading states:

```tsx
function Dashboard() {
  return (
    <Suspense fallback={<PageSkeleton />}>
      <Header />
      <main>
        <Suspense fallback={<SidebarSkeleton />}>
          <Sidebar />
        </Suspense>
        <Suspense fallback={<ContentSkeleton />}>
          <MainContent />
        </Suspense>
      </main>
    </Suspense>
  );
}
```

## Revealing Content Together

Wrap related content in a single Suspense to reveal them together:

```tsx
// Shows skeleton until BOTH are ready
<Suspense fallback={<ProfileSkeleton />}>
  <ProfileHeader user={user} />
  <ProfileStats user={user} />
</Suspense>

// vs. showing each independently
<Suspense fallback={<HeaderSkeleton />}>
  <ProfileHeader user={user} />
</Suspense>
<Suspense fallback={<StatsSkeleton />}>
  <ProfileStats user={user} />
</Suspense>
```

## Suspense with Transitions

Avoid showing fallbacks for already-visible content:

```tsx
function TabContainer() {
  const [tab, setTab] = useState('home');
  const [isPending, startTransition] = useTransition();

  const selectTab = (nextTab) => {
    startTransition(() => {
      setTab(nextTab);
    });
  };

  return (
    <>
      <TabButtons onSelect={selectTab} isPending={isPending} />
      <div style={{ opacity: isPending ? 0.7 : 1 }}>
        <Suspense fallback={<TabSkeleton />}>
          {tab === 'home' && <Home />}
          {tab === 'posts' && <Posts />}
        </Suspense>
      </div>
    </>
  );
}
```

With `startTransition`, React keeps showing the old tab (dimmed) instead of the fallback.

## Server Components & Suspense

Suspense boundaries define where streaming happens:

```tsx
// Server Component
async function Page() {
  return (
    <>
      <Header /> {/* Sent immediately */}
      <Suspense fallback={<PostsSkeleton />}>
        <Posts /> {/* Streamed when ready */}
      </Suspense>
    </>
  );
}
```

## Suspense-Enabled Data Fetching

Modern data fetching libraries integrate with Suspense:

```tsx
// With React Query / TanStack Query
function UserProfile({ userId }) {
  const { data: user } = useSuspenseQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId)
  });
  
  return <Profile user={user} />;
}

// Wrap in Suspense
<Suspense fallback={<ProfileSkeleton />}>
  <UserProfile userId={123} />
</Suspense>
```

## Resources

- [Suspense API Reference](https://react.dev/reference/react/Suspense) — Official React documentation for Suspense

---

> 📘 *This lesson is part of the [React Advanced APIs](https://stanza.dev/courses/react-advanced-apis) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*