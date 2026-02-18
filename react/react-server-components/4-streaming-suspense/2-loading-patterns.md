---
source_course: "react-server-components"
source_lesson: "react-server-components-loading-patterns"
---

# Loading State Patterns

Master different approaches to handling loading states in RSC applications.

## Pattern 1: Instant Loading States

Show loading UI immediately while content loads:

```tsx
// app/posts/loading.tsx
export default function Loading() {
  return (
    <div className="posts-loading">
      {[1, 2, 3].map(i => (
        <PostSkeleton key={i} />
      ))}
    </div>
  );
}

// app/posts/page.tsx
async function PostsPage() {
  const posts = await getPosts(); // Takes 2 seconds
  return <PostList posts={posts} />;
}

// User sees: loading.tsx immediately, then page.tsx after 2s
```

## Pattern 2: Streaming Sections

Stream independent sections as they're ready:

```tsx
async function ProductPage({ id }) {
  return (
    <div>
      {/* Fast - renders immediately */}
      <Suspense fallback={<ProductSkeleton />}>
        <ProductDetails id={id} />
      </Suspense>
      
      {/* Medium speed */}
      <Suspense fallback={<ReviewsSkeleton />}>
        <ProductReviews id={id} />
      </Suspense>
      
      {/* Slow - doesn't block others */}
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations id={id} />
      </Suspense>
    </div>
  );
}
```

## Pattern 3: Grouped Loading

Load related content together:

```tsx
async function UserProfile({ userId }) {
  return (
    <div>
      {/* These load together */}
      <Suspense fallback={<ProfileSkeleton />}>
        <ProfileHeader userId={userId} />
        <ProfileStats userId={userId} />
        <ProfileBio userId={userId} />
      </Suspense>
      
      {/* This loads independently */}
      <Suspense fallback={<ActivitySkeleton />}>
        <RecentActivity userId={userId} />
      </Suspense>
    </div>
  );
}
```

## Pattern 4: Staggered Loading

Control the order content appears:

```tsx
async function Feed() {
  return (
    <div>
      {/* Priority content - no Suspense, blocks render */}
      <FeaturedPost />
      
      {/* Secondary content - streams in */}
      <Suspense fallback={<FeedSkeleton />}>
        <FeedPosts />
      </Suspense>
      
      {/* Tertiary content - loads last */}
      <Suspense fallback={<SidebarSkeleton />}>
        <Sidebar />
      </Suspense>
    </div>
  );
}
```

## Pattern 5: Optimistic Navigation

Keep showing old content during navigation:

```tsx
'use client';

import { useTransition } from 'react';
import { useRouter } from 'next/navigation';

function Navigation() {
  const router = useRouter();
  const [isPending, startTransition] = useTransition();
  
  const navigate = (path) => {
    startTransition(() => {
      router.push(path);
    });
  };
  
  return (
    <nav style={{ opacity: isPending ? 0.7 : 1 }}>
      <button onClick={() => navigate('/home')}>Home</button>
      <button onClick={() => navigate('/about')}>About</button>
      {isPending && <Spinner />}
    </nav>
  );
}
```

## Avoiding Loading State Pitfalls

### Don't Over-Suspend

```tsx
// ❌ Too many boundaries - jarring experience
<Suspense fallback={<Skeleton />}>
  <Title />
</Suspense>
<Suspense fallback={<Skeleton />}>
  <Subtitle />
</Suspense>
<Suspense fallback={<Skeleton />}>
  <Paragraph />
</Suspense>

// ✅ Group related content
<Suspense fallback={<ArticleSkeleton />}>
  <Title />
  <Subtitle />
  <Paragraph />
</Suspense>
```

### Match Skeleton to Content

```tsx
// ❌ Generic skeleton doesn't match content
<Suspense fallback={<Spinner />}>
  <ComplexDataTable />
</Suspense>

// ✅ Skeleton matches the actual layout
<Suspense fallback={<DataTableSkeleton columns={5} rows={10} />}>
  <ComplexDataTable />
</Suspense>
```

## Resources

- [Loading UI and Streaming](https://react.dev/reference/react/Suspense) — React Suspense documentation

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*