---
source_course: "react-server-components"
source_lesson: "react-server-components-data-patterns"
---

# Data Fetching Patterns in RSC

Learn the best patterns for fetching and managing data in Server Components.

## Pattern 1: Fetch Where You Need It

Fetch data in the component that uses it:

```tsx
// Each component fetches its own data
async function Page({ productId }) {
  return (
    <div>
      <ProductDetails productId={productId} />
      <ProductReviews productId={productId} />
      <RelatedProducts productId={productId} />
    </div>
  );
}

async function ProductDetails({ productId }) {
  const product = await getProduct(productId);
  return <div>{product.name}</div>;
}

async function ProductReviews({ productId }) {
  const reviews = await getReviews(productId);
  return <ReviewList reviews={reviews} />;
}
```

## Pattern 2: Preload Pattern

Start fetching before you need the data:

```tsx
import { preload } from './data';

async function Page({ productId }) {
  // Start fetching immediately
  preload(productId);
  
  return (
    <div>
      <Suspense fallback={<Skeleton />}>
        <ProductDetails productId={productId} />
      </Suspense>
    </div>
  );
}

// data.ts
const cache = new Map();

export function preload(productId) {
  if (!cache.has(productId)) {
    cache.set(productId, fetchProduct(productId));
  }
}

export async function getProduct(productId) {
  if (!cache.has(productId)) {
    cache.set(productId, fetchProduct(productId));
  }
  return cache.get(productId);
}
```

## Pattern 3: Passing Data Down

Fetch once, pass to children:

```tsx
async function ProductPage({ productId }) {
  // Single fetch at the top
  const product = await getProduct(productId);
  
  return (
    <div>
      <ProductHeader product={product} />
      <ProductGallery images={product.images} />
      <ProductInfo product={product} />
      <AddToCart product={product} /> {/* Client Component */}
    </div>
  );
}
```

## Pattern 4: Parallel Routes (Next.js)

Load independent sections in parallel:

```tsx
// app/dashboard/layout.tsx
export default function Layout({ children, analytics, team }) {
  return (
    <div>
      {children}
      <div className="sidebar">
        {analytics} {/* @analytics/page.tsx */}
        {team}      {/* @team/page.tsx */}
      </div>
    </div>
  );
}

// app/dashboard/@analytics/page.tsx
async function Analytics() {
  const data = await getAnalytics(); // Fetches in parallel
  return <AnalyticsChart data={data} />;
}

// app/dashboard/@team/page.tsx
async function Team() {
  const members = await getTeamMembers(); // Fetches in parallel
  return <TeamList members={members} />;
}
```

## Error Handling

```tsx
async function ProductPage({ productId }) {
  const product = await getProduct(productId);
  
  if (!product) {
    notFound(); // Next.js: shows 404 page
  }
  
  return <Product product={product} />;
}

// Or with try/catch
async function SafeProductPage({ productId }) {
  try {
    const product = await getProduct(productId);
    return <Product product={product} />;
  } catch (error) {
    return <ErrorMessage error={error} />;
  }
}
```

## Revalidation Strategies

```tsx
// Time-based revalidation (Next.js)
export const revalidate = 3600; // Revalidate every hour

async function BlogPosts() {
  const posts = await getPosts();
  return <PostList posts={posts} />;
}

// On-demand revalidation
import { revalidatePath, revalidateTag } from 'next/cache';

async function updatePost(id, data) {
  await db.posts.update({ where: { id }, data });
  revalidatePath('/blog');
  revalidateTag('posts');
}
```

## Resources

- [Data Fetching Patterns](https://react.dev/learn/synchronizing-with-effects#fetching-data) — React documentation on data fetching

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*