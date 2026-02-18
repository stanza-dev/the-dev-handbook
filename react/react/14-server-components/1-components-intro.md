---
source_course: "react"
source_lesson: "react-server-components-intro"
---

# React Server Components

Server Components execute on the server and send rendered HTML to the client. They never download to the browser.

## Advantages

- **Smaller bundles**: Server-only code stays on server
- **Direct data access**: Query databases, read files directly
- **Secure**: API keys and sensitive logic stay server-side
- **SEO-friendly**: Full HTML sent to crawlers

## Distinguishing Server vs Client

| Aspect | Server Component | Client Component |
|--------|------------------|------------------|
| Directive | None (default) | `'use client'` |
| Where runs | Server only | Browser |
| Can use hooks | No | Yes |
| Can use browser APIs | No | Yes |
| Can be async | Yes | No |
| Can access backend | Directly | Via API |

## Composition Pattern

Server Components can import Client Components, but not vice versa. Pass server data as props:

## Code Examples

**Server Component with Client Component children**

```tsx
// ProductPage.tsx - Server Component (no directive)
import { getProduct, getReviews } from '@/db';
import { AddToCartButton } from './AddToCartButton';
import { ReviewList } from './ReviewList';

export default async function ProductPage({ id }: { id: string }) {
  // Direct database access - this code never runs in browser
  const product = await getProduct(id);
  const reviews = await getReviews(id);
  
  return (
    <article>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p className="price">${product.price}</p>
      
      {/* Client Component for interactivity */}
      <AddToCartButton productId={id} />
      
      {/* Server Component can render other Server Components */}
      <ReviewList reviews={reviews} />
    </article>
  );
}

// AddToCartButton.tsx - Client Component
'use client';

import { useState } from 'react';

export function AddToCartButton({ productId }: { productId: string }) {
  const [isAdding, setIsAdding] = useState(false);
  
  async function handleAdd() {
    setIsAdding(true);
    await addToCart(productId);
    setIsAdding(false);
  }
  
  return (
    <button onClick={handleAdd} disabled={isAdding}>
      {isAdding ? 'Adding...' : 'Add to Cart'}
    </button>
  );
}
```


## Resources

- [Server Components](https://react.dev/reference/rsc/server-components) — Official Server Components reference

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*