---
source_course: "react-intermediate"
source_lesson: "react-preventing-rerenders"
---

# Preventing Unnecessary Re-renders

## Introduction

Once you've identified unnecessary re-renders through profiling, React provides several tools to prevent them: `memo`, `useMemo`, and `useCallback`.

## Key Concepts

**Memoization** caches results so they don't need to be recalculated. In React, we memoize components, values, and functions to skip work when inputs haven't changed.

## Real World Context

Strategic memoization is crucial for:
- Large lists where most items don't change
- Expensive computations (filtering, sorting, transformations)
- Components with stable props passed to memoized children
- Preventing cascade re-renders in deep trees

## Deep Dive

### React.memo for Components

Memo wraps a component to skip re-renders when props are equal:

```tsx
const ExpensiveChart = memo(function ExpensiveChart({ data }) {
  // Only re-renders when data actually changes
  return <Chart data={data} />;
});
```

**Important**: Memo uses shallow comparison. Objects/arrays must be referentially stable:

```tsx
// ❌ New array every render - memo is useless
<ExpensiveChart data={items.filter(i => i.active)} />

// ✅ Memoize the filtered array
const activeItems = useMemo(
  () => items.filter(i => i.active),
  [items]
);
<ExpensiveChart data={activeItems} />
```

### useMemo for Expensive Calculations

```tsx
function SearchResults({ query, items }) {
  // Only recalculate when query or items change
  const filteredItems = useMemo(() => {
    return items.filter(item =>
      item.name.toLowerCase().includes(query.toLowerCase())
    );
  }, [query, items]);
  
  return <List items={filteredItems} />;
}
```

### useCallback for Stable Functions

```tsx
function Parent({ items }) {
  // Without useCallback: new function every render
  // Child re-renders even with memo
  
  // With useCallback: same function reference
  const handleSelect = useCallback((id: string) => {
    console.log('Selected:', id);
  }, []); // No deps = never changes
  
  return items.map(item => (
    <MemoizedItem
      key={item.id}
      item={item}
      onSelect={handleSelect}
    />
  ));
}
```

### When Memo Doesn't Help

1. **Props always change**: New objects/arrays every render
2. **Children prop changes**: Inline JSX creates new children
3. **Context changes**: Memo doesn't block context updates
4. **Render is cheap**: Memo overhead exceeds render cost

### Custom Comparison

```tsx
const Chart = memo(
  function Chart({ data, options }) {
    return /* ... */;
  },
  (prevProps, nextProps) => {
    // Return true to skip re-render
    return (
      prevProps.data.length === nextProps.data.length &&
      prevProps.options.theme === nextProps.options.theme
    );
  }
);
```

## Common Pitfalls

1. **Memoizing everything**: Adds memory and comparison overhead. Only memoize what's actually slow.
2. **Missing dependencies**: Wrong deps array causes stale values or infinite loops.
3. **Object/array props without useMemo**: Memo is useless if you pass new objects every render.

## Best Practices

- Profile first - don't guess what's slow
- Start with `memo` on expensive components
- Add `useMemo`/`useCallback` when needed to stabilize props
- Keep dependency arrays minimal and correct
- Consider component composition over memoization
- Don't memoize cheap operations

## Summary

Use `memo` for components, `useMemo` for values, and `useCallback` for functions. Always profile first. Memoization only helps when props are actually stable - use `useMemo` to create stable object/array props.

## Code Examples

**Complete memoization example with memo, useMemo, and useCallback**

```tsx
import { memo, useMemo, useCallback, useState } from 'react';

// Memoized list item
const ProductCard = memo(function ProductCard({
  product,
  onAddToCart
}: {
  product: Product;
  onAddToCart: (id: string) => void;
}) {
  console.log(`ProductCard ${product.id} rendered`);
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3>
      <p>${product.price}</p>
      <button onClick={() => onAddToCart(product.id)}>
        Add to Cart
      </button>
    </div>
  );
});

function ProductList({ products, category }: {
  products: Product[];
  category: string;
}) {
  const [cart, setCart] = useState<string[]>([]);
  const [sortBy, setSortBy] = useState<'name' | 'price'>('name');

  // Memoize filtered products
  const filteredProducts = useMemo(() => {
    return products.filter(p => p.category === category);
  }, [products, category]);

  // Memoize sorted products
  const sortedProducts = useMemo(() => {
    return [...filteredProducts].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      return a.price - b.price;
    });
  }, [filteredProducts, sortBy]);

  // Stable callback for memoized children
  const handleAddToCart = useCallback((id: string) => {
    setCart(prev => [...prev, id]);
  }, []);

  return (
    <div>
      <div>Cart: {cart.length} items</div>
      <select
        value={sortBy}
        onChange={(e) => setSortBy(e.target.value as 'name' | 'price')}
      >
        <option value="name">Sort by Name</option>
        <option value="price">Sort by Price</option>
      </select>
      
      <div className="product-grid">
        {sortedProducts.map(product => (
          <ProductCard
            key={product.id}
            product={product}
            onAddToCart={handleAddToCart}
          />
        ))}
      </div>
    </div>
  );
}
```


## Resources

- [memo Reference](https://react.dev/reference/react/memo) — Official React.memo documentation
- [useMemo Reference](https://react.dev/reference/react/useMemo) — Official useMemo documentation

---

> 📘 *This lesson is part of the [React Intermediate](https://stanza.dev/courses/react-intermediate) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*