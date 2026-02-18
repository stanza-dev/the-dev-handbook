---
source_course: "react-server-components"
source_lesson: "react-server-components-streaming"
---

# Streaming: Progressive Page Loading

Streaming allows you to send HTML to the client progressively, showing content as it becomes ready rather than waiting for everything.

## How Streaming Works

1. **Server starts rendering** - Begins processing the React tree
2. **Shell is ready** - Static parts (layout, navigation) are sent immediately
3. **Content streams in** - Dynamic parts are sent as they complete
4. **Client hydrates** - JavaScript makes the page interactive

## The Shell

The "shell" is everything outside Suspense boundaries:

```tsx
async function Page() {
  return (
    <html>
      <body>
        {/* Shell - sent immediately */}
        <Header />
        <nav>Navigation</nav>
        
        {/* Streamed content */}
        <Suspense fallback={<MainSkeleton />}>
          <MainContent />
        </Suspense>
        
        {/* Shell - sent immediately */}
        <Footer />
      </body>
    </html>
  );
}
```

## Streaming Timeline

```
Time 0ms:    Shell sent (Header, Nav, Skeletons, Footer)
Time 100ms:  <UserProfile /> streams in, replaces skeleton
Time 300ms:  <RecentPosts /> streams in, replaces skeleton
Time 500ms:  <Recommendations /> streams in, replaces skeleton
```

## Nested Suspense for Granular Loading

```tsx
async function Dashboard() {
  return (
    <div className="dashboard">
      {/* Level 1: Main layout */}
      <Suspense fallback={<DashboardSkeleton />}>
        <DashboardLayout>
          {/* Level 2: Independent sections */}
          <Suspense fallback={<StatsSkeleton />}>
            <Stats />
          </Suspense>
          
          <Suspense fallback={<ChartSkeleton />}>
            <AnalyticsChart />
          </Suspense>
          
          <Suspense fallback={<TableSkeleton />}>
            <RecentActivity />
          </Suspense>
        </DashboardLayout>
      </Suspense>
    </div>
  );
}
```

## Loading UI Patterns

### Skeleton Screens

```tsx
function PostSkeleton() {
  return (
    <div className="post-skeleton">
      <div className="skeleton-title" />
      <div className="skeleton-line" />
      <div className="skeleton-line" />
      <div className="skeleton-line short" />
    </div>
  );
}

<Suspense fallback={<PostSkeleton />}>
  <Post id={postId} />
</Suspense>
```

### Instant Loading States (Next.js)

```tsx
// app/dashboard/loading.tsx
export default function Loading() {
  return <DashboardSkeleton />;
}

// Automatically wraps page.tsx in Suspense
```

## Benefits of Streaming

1. **Faster Time to First Byte (TTFB)** - Don't wait for slow data
2. **Better perceived performance** - Users see content sooner
3. **Improved Core Web Vitals** - Better LCP scores
4. **Graceful degradation** - Slow components don't block fast ones

## Streaming vs Traditional SSR

| Aspect | Traditional SSR | Streaming SSR |
|--------|-----------------|---------------|
| First content | After all data loads | Immediately |
| Slow queries | Block everything | Only block their section |
| User experience | All or nothing | Progressive |
| TTFB | Slow | Fast |

## Resources

- [Streaming with Suspense](https://react.dev/reference/react/Suspense#revealing-content-together-at-once) — React documentation on Suspense and streaming

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*