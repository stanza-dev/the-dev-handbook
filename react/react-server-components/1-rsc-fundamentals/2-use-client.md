---
source_course: "react-server-components"
source_lesson: "react-server-components-use-client"
---

# 'use client': Opting Into Interactivity

The `'use client'` directive marks a component (and everything it imports) as a Client Component. Use it when you need interactivity, state, or browser APIs.

## Basic Usage

```tsx
'use client';

import { useState } from 'react';

export function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      Count: {count}
    </button>
  );
}
```

## When You Need 'use client'

Add `'use client'` when your component uses:

### 1. React Hooks

```tsx
'use client';

import { useState, useEffect, useRef } from 'react';

function SearchInput() {
  const [query, setQuery] = useState('');
  const inputRef = useRef(null);
  
  useEffect(() => {
    inputRef.current?.focus();
  }, []);
  
  return <input ref={inputRef} value={query} onChange={...} />;
}
```

### 2. Event Handlers

```tsx
'use client';

function LikeButton({ postId }) {
  const handleClick = async () => {
    await likePost(postId);
  };
  
  return <button onClick={handleClick}>❤️ Like</button>;
}
```

### 3. Browser APIs

```tsx
'use client';

import { useEffect, useState } from 'react';

function WindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    handleResize();
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return <p>{size.width} x {size.height}</p>;
}
```

## The Boundary Effect

When you add `'use client'`, everything imported into that file becomes part of the client bundle:

```tsx
'use client';

// All of these become client code:
import { useState } from 'react';
import { Button } from './Button';      // Now a Client Component
import { formatDate } from './utils';   // Included in client bundle
import { motion } from 'framer-motion'; // Included in client bundle
```

## Placement Strategy

Push `'use client'` as far down the tree as possible:

```tsx
// ❌ Bad - entire page is a Client Component
'use client';

function ProductPage({ product }) {
  const [quantity, setQuantity] = useState(1);
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p> {/* Static - doesn't need client */}
      <Reviews productId={product.id} /> {/* Could be Server */}
      <QuantitySelector value={quantity} onChange={setQuantity} />
    </div>
  );
}
```

```tsx
// ✅ Good - only interactive parts are Client Components

// ProductPage.tsx (Server Component)
async function ProductPage({ id }) {
  const product = await getProduct(id);
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <Reviews productId={id} />
      <AddToCart productId={id} /> {/* Only this needs 'use client' */}
    </div>
  );
}

// AddToCart.tsx
'use client';

function AddToCart({ productId }) {
  const [quantity, setQuantity] = useState(1);
  // ...
}
```

## Resources

- ['use client' Directive](https://react.dev/reference/rsc/use-client) — Official React documentation for the 'use client' directive

---

> 📘 *This lesson is part of the [React Server Components](https://stanza.dev/courses/react-server-components) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*