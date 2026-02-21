---
source_course: "react-modern-patterns"
source_lesson: "react-server-component-patterns"
---

# Server Component Patterns

## Introduction

Now that you understand the basics of Server Components and the server-client boundary, it is time to learn the practical patterns that make Server Components shine in real applications. These patterns cover data fetching, streaming, parallel loading, and how to structure your component tree for maximum performance. Mastering these patterns will help you build applications that are both fast and maintainable.

## Key Concepts

- **Async Components**: Server Components can be `async` functions that `await` data directly in their body, eliminating the need for useEffect-based data fetching.
- **Streaming**: Server Components can stream their output progressively. Components wrapped in `<Suspense>` can render independently, showing fallbacks for slow parts while fast parts display immediately.
- **Parallel Data Fetching**: By starting multiple database queries or API calls simultaneously with `Promise.all`, you avoid sequential waterfalls.
- **Component-Level Caching**: Frameworks like Next.js cache the output of Server Components, so repeated visits to the same page avoid redundant computation.

## Real World Context

Imagine a news homepage with a hero article, trending stories, weather widget, and stock ticker. Each section fetches data from different sources at different speeds. With Server Components and Suspense streaming, the hero article (fast database query) renders immediately, while the weather widget and stock ticker (slow external APIs) show skeleton loaders. As each slow section resolves, it streams in without blocking the rest of the page.

## Deep Dive

Parallel data fetching avoids sequential waterfalls:

```tsx
// Bad: Sequential — each await blocks the next
async function DashboardSlow() {
  const user = await getUser();         // 200ms
  const posts = await getPosts();       // 300ms
  const analytics = await getAnalytics(); // 400ms
  // Total: ~900ms

  return <Dashboard user={user} posts={posts} analytics={analytics} />;
}

// Good: Parallel — all requests start simultaneously
async function DashboardFast() {
  const [user, posts, analytics] = await Promise.all([
    getUser(),        // 200ms
    getPosts(),       // 300ms
    getAnalytics(),   // 400ms
  ]);
  // Total: ~400ms (slowest request)

  return <Dashboard user={user} posts={posts} analytics={analytics} />;
}
```

Streaming with Suspense lets you progressively render sections:

```tsx
import { Suspense } from "react";

async function HeroArticle() {
  const article = await db.articles.findFirst({ orderBy: { createdAt: "desc" } });
  return <article><h1>{article.title}</h1><p>{article.summary}</p></article>;
}

async function TrendingStories() {
  const stories = await fetchTrending(); // Slow external API
  return <ul>{stories.map(s => <li key={s.id}>{s.title}</li>)}</ul>;
}

export default function Homepage() {
  return (
    <div>
      <Suspense fallback={<HeroSkeleton />}>
        <HeroArticle />
      </Suspense>
      <Suspense fallback={<TrendingSkeleton />}>
        <TrendingStories />
      </Suspense>
    </div>
  );
}
```

Another powerful pattern is the data preloading pattern where you start fetching in a parent and pass the Promise down:

```tsx
async function Page({ id }: { id: string }) {
  // Start fetch immediately, don't await
  const dataPromise = fetchExpensiveData(id);

  return (
    <Suspense fallback={<Loading />}>
      <DataConsumer dataPromise={dataPromise} />
    </Suspense>
  );
}
```

## Common Pitfalls

1. **Sequential awaits when parallel is possible** — Each `await` in a Server Component blocks the next. If two pieces of data are independent, use `Promise.all` to fetch them simultaneously.
2. **Over-suspending** — Wrapping every tiny component in its own `<Suspense>` boundary creates a jarring loading experience with too many independent spinners. Group related content under a single boundary.

## Best Practices

1. **Use `Promise.all` for independent data** — When a component needs data from multiple sources that do not depend on each other, fetch them in parallel to reduce total loading time.
2. **Place Suspense boundaries at meaningful UI sections** — Wrap logical sections of your page (hero, sidebar, comments) rather than individual components. This creates a smoother progressive loading experience.

## Summary

- Server Components can be async functions that await data directly, eliminating useEffect waterfalls.
- Use `Promise.all` for independent data fetches and Suspense boundaries for progressive streaming.
- Place Suspense boundaries around meaningful UI sections rather than individual components for a smooth user experience.

## Code Examples

**Parallel fetching and Suspense streaming in Server Components**

```tsx
import { Suspense } from "react";

// Parallel data fetching in a Server Component
async function Dashboard() {
  const [user, posts, stats] = await Promise.all([
    getUser(),
    getPosts(),
    getStats(),
  ]);

  return (
    <div>
      <h1>Welcome, {user.name}</h1>
      <PostList posts={posts} />
      <StatsPanel stats={stats} />
    </div>
  );
}

// Streaming with Suspense
export default function Page() {
  return (
    <div>
      <Suspense fallback={<DashboardSkeleton />}>
        <Dashboard />
      </Suspense>
      <Suspense fallback={<RecommendationsSkeleton />}>
        <Recommendations />
      </Suspense>
    </div>
  );
}
```


## Resources

- [Streaming with Suspense](https://react.dev/reference/react/Suspense) — Official Suspense documentation for streaming rendering

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*