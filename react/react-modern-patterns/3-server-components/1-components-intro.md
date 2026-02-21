---
source_course: "react-modern-patterns"
source_lesson: "react-server-components-intro"
---

# Server Components Explained

## Introduction

React Server Components represent a fundamental shift in how we think about component architecture. Instead of shipping all your component code to the browser, Server Components execute entirely on the server, send their rendered output as HTML and a special serialized format, and never appear in the client JavaScript bundle. This means faster page loads, smaller bundles, and direct access to server-side resources like databases and file systems.

## Key Concepts

- **Server Component**: A React component that runs exclusively on the server. It is the default component type in frameworks like Next.js App Router (no directive needed).
- **Client Component**: A component marked with the `"use client"` directive that runs in the browser and can use hooks, event handlers, and browser APIs.
- **Serialization Boundary**: The point where Server Components pass data to Client Components via props. Props must be serializable (no functions, classes, or Dates without conversion).
- **Zero Bundle Cost**: Code in Server Components, including their imports, is never sent to the browser, meaning heavy libraries used only for rendering stay server-side.

## Real World Context

Consider an e-commerce product page. The product description, pricing, reviews, and recommendations all come from a database. With traditional Client Components, you would need API routes, fetch calls, loading states, and error handling on the client. With Server Components, you query the database directly inside the component, render the HTML on the server, and send it to the browser. The only JavaScript sent to the client is for interactive elements like the "Add to Cart" button.

## Deep Dive

Server Components are the default in React 19 when used with a compatible framework. You do not add any directive:

```tsx
// ProductPage.tsx — This is a Server Component (no directive)
import { db } from "@/lib/database";
import { AddToCartButton } from "./AddToCartButton";

export default async function ProductPage({ id }: { id: string }) {
  // Direct database access — this never runs in the browser
  const product = await db.product.findUnique({ where: { id } });
  const reviews = await db.review.findMany({ where: { productId: id } });

  return (
    <article>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p className="price">${product.price.toFixed(2)}</p>

      {/* Client Component for interactivity */}
      <AddToCartButton productId={id} price={product.price} />

      <section>
        <h2>Reviews ({reviews.length})</h2>
        {reviews.map((review) => (
          <div key={review.id}>
            <strong>{review.author}</strong>
            <p>{review.text}</p>
          </div>
        ))}
      </section>
    </article>
  );
}
```

```tsx
// AddToCartButton.tsx — Client Component
"use client";

import { useState } from "react";

export function AddToCartButton({ productId, price }: { productId: string; price: number }) {
  const [added, setAdded] = useState(false);

  return (
    <button onClick={() => { addToCart(productId); setAdded(true); }}>
      {added ? "Added!" : `Add to Cart - $${price.toFixed(2)}`}
    </button>
  );
}
```

Here is the corrected comparison between Server and Client Components:

| Aspect | Server Component | Client Component |
|--------|-----------------|-----------------|
| Directive | None (default) | `"use client"` |
| Runs on | Server only | Server (SSR) + Browser |
| Can use useState/useEffect | No | Yes |
| Can use browser APIs | No | Yes |
| Can be async | Yes | Yes (via useTransition for async operations) |
| Can access backend directly | Yes | Via API calls |
| In JS bundle | No | Yes |

Note: Client Components in React 19 can perform async operations using `useTransition` with async functions. The distinction is that Server Components can be declared as `async function` components that `await` directly in their body, while Client Components handle async work through hooks and transitions.

## Common Pitfalls

1. **Trying to use hooks in Server Components** — Server Components run once on the server and produce static output. They cannot use `useState`, `useEffect`, or any hook that depends on client-side re-rendering. If you need interactivity, extract that part into a Client Component.
2. **Passing non-serializable props across the boundary** — When a Server Component renders a Client Component, all props must be serializable (strings, numbers, plain objects, arrays). You cannot pass functions, class instances, or Map/Set objects as props.

## Best Practices

1. **Keep Client Components at the leaves** — Push `"use client"` boundaries as deep as possible in your component tree. Only the interactive parts (buttons, forms, animations) need to be Client Components, keeping the majority of your tree server-rendered.
2. **Use Server Components for data fetching** — Instead of fetching data on the client with useEffect, fetch it in a Server Component and pass the data as props to Client Components. This eliminates waterfalls and reduces client JavaScript.

## Summary

- Server Components run on the server, have zero bundle cost, and can directly access databases and file systems.
- Client Components are marked with `"use client"` and handle interactivity with hooks and browser APIs.
- Props passed from Server to Client Components must be serializable; keep Client Components at the leaf nodes of your tree for optimal performance.

## Code Examples

**Server Component fetching data and rendering a Client Component child**

```tsx
// Server Component — no directive needed, runs on server only
import { db } from "@/lib/database";
import { LikeButton } from "./LikeButton";

export default async function PostPage({ id }: { id: string }) {
  const post = await db.post.findUnique({ where: { id } });

  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
      <LikeButton postId={id} initialCount={post.likes} />
    </article>
  );
}

// Client Component — "use client" directive
"use client";
import { useState } from "react";

export function LikeButton({ postId, initialCount }: { postId: string; initialCount: number }) {
  const [count, setCount] = useState(initialCount);
  return <button onClick={() => setCount(c => c + 1)}>Likes: {count}</button>;
}
```


## Resources

- [Server Components](https://react.dev/reference/rsc/server-components) — Official Server Components reference

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*