---
source_course: "react-data-fetching"
source_lesson: "react-data-fetching-suspense-patterns"
---

# Suspense Patterns

Advanced patterns for using Suspense effectively.

## Fetch-Then-Render

Start fetching before rendering:

```jsx
// Start fetch immediately when user clicks
function ProfilePage() {
  const [resource, setResource] = useState(null);

  function handleClick(userId) {
    // Start fetch immediately
    setResource(fetchProfileData(userId));
  }

  return (
    <div>
      <button onClick={() => handleClick('123')}>Load Profile</button>
      {resource && (
        <Suspense fallback={<Loading />}>
          <ProfileDetails resource={resource} />
        </Suspense>
      )}
    </div>
  );
}
```

## Render-as-You-Fetch

Start fetch before navigation:

```jsx
// In router or link handler
function handleNavigation(userId) {
  // Start fetching before navigation
  const resource = fetchProfileData(userId);
  navigate(`/profile/${userId}`, { state: { resource } });
}

// In ProfilePage
function ProfilePage() {
  const { resource } = useLocation().state;
  
  return (
    <Suspense fallback={<Loading />}>
      <Profile resource={resource} />
    </Suspense>
  );
}
```

## Parallel Data Fetching

Fetch independent data in parallel:

```jsx
function Dashboard() {
  return (
    <div className="dashboard">
      {/* These fetch in parallel */}
      <Suspense fallback={<UserSkeleton />}>
        <UserPanel />
      </Suspense>
      
      <Suspense fallback={<StatsSkeleton />}>
        <StatsPanel />
      </Suspense>
      
      <Suspense fallback={<ActivitySkeleton />}>
        <ActivityFeed />
      </Suspense>
    </div>
  );
}
```

## SuspenseList (Experimental)

Coordinate multiple suspense boundaries:

```jsx
<SuspenseList revealOrder="forwards">
  <Suspense fallback={<ItemSkeleton />}>
    <Item id={1} />
  </Suspense>
  <Suspense fallback={<ItemSkeleton />}>
    <Item id={2} />
  </Suspense>
  <Suspense fallback={<ItemSkeleton />}>
    <Item id={3} />
  </Suspense>
</SuspenseList>
```

## Avoiding Waterfalls

```jsx
// ❌ Bad: Waterfall (sequential)
function Profile({ userId }) {
  const user = use(fetchUser(userId));    // Wait...
  const posts = use(fetchPosts(userId));  // Then wait...
  return <ProfileView user={user} posts={posts} />;
}

// ✅ Good: Parallel fetching
function Profile({ userId }) {
  // Start both fetches
  const userPromise = fetchUser(userId);
  const postsPromise = fetchPosts(userId);
  
  // Then use both
  const user = use(userPromise);
  const posts = use(postsPromise);
  
  return <ProfileView user={user} posts={posts} />;
}
```

---

> 📘 *This lesson is part of the [React Data Fetching Patterns](https://stanza.dev/courses/react-data-fetching) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*