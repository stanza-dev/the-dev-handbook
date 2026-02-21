---
source_course: "react-modern-patterns"
source_lesson: "react-state-management-patterns"
---

# State Management Patterns

## Introduction

Choosing the right state management approach is one of the most impactful architectural decisions in a React application. This lesson covers when to use local state, Context, and external libraries, and how to avoid common over-engineering mistakes.

## Key Concepts

- **Local state (useState/useReducer)**: State that belongs to a single component or a small subtree. The default and simplest option.
- **Context API**: Built-in React mechanism for sharing state across a subtree without prop drilling. Best for infrequently-changing values.
- **External state libraries**: Tools like Zustand, Jotai, or Redux for complex state that needs to be shared globally with fine-grained subscriptions.
- **Server state**: Data from the server managed by libraries like TanStack Query. Different from client UI state.

## Real World Context

A team uses Redux for everything: form inputs, modal visibility, selected tabs, user authentication, and cached API data. The Redux store has 50 slices, actions take 3 files to create, and simple UI changes require touching multiple files. Refactoring to use local state for UI concerns, TanStack Query for server data, and a small Zustand store for global client state reduces complexity dramatically.

## Deep Dive

**Decision framework for state placement:**

| State Type | Solution | Examples |
|-----------|----------|---------|
| Component UI state | useState | Form inputs, toggles, local filters |
| Complex local logic | useReducer | Multi-step forms, state machines |
| Subtree shared state | Context | Theme, locale, auth user |
| Global client state | Zustand/Jotai | Shopping cart, notifications |
| Server data | TanStack Query/SWR | API responses, cached data |
| URL state | useSearchParams/router | Filters, pagination, tabs |

**Keep state local by default:**

```tsx
// Only lift state when truly needed
function SearchPage() {
  return (
    <div>
      <SearchBar />    {/* Manages its own input state */}
      <FilterPanel />  {/* Manages its own filter state */}
      <ResultsList />  {/* Gets data from TanStack Query */}
    </div>
  );
}
```

**Context for infrequent updates:**

```tsx
// Good: theme changes rarely
const ThemeContext = createContext<'light' | 'dark'>('light');

// Bad: rapidly changing values cause all consumers to re-render
const MousePositionContext = createContext({ x: 0, y: 0 });
```

**Separate server state from client state.** Server state has different concerns: caching, background refresh, invalidation, optimistic updates. Let TanStack Query handle it instead of mixing it into your client state store.

## Common Pitfalls

1. **Putting everything in global state** — Most state is local. Only lift or globalize state when multiple unrelated components need it.
2. **Using Context for rapidly changing values** — Context causes all consumers to re-render on every change. For frequent updates, use an external library with selectors.

## Best Practices

1. **Start local, lift as needed** — Begin with useState. Only move state up or into Context/stores when you have a concrete need.
2. **Separate server state from client state** — Use TanStack Query for server data and local state or Zustand for client-only concerns.

## Summary

- Default to local state; lift only when needed.
- Use Context for infrequently changing shared values (theme, auth, locale).
- Use external libraries for complex global state with frequent updates.
- Keep server state in a dedicated library like TanStack Query.

## Code Examples

**State placement: server state vs local UI state**

```tsx
// State placement decision example
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

function ProductPage({ productId }: { productId: string }) {
  // Server state: TanStack Query
  const { data: product } = useQuery({
    queryKey: ['product', productId],
    queryFn: () => fetchProduct(productId),
  });

  // Local UI state: useState
  const [selectedSize, setSelectedSize] = useState('M');
  const [showReviews, setShowReviews] = useState(false);

  // Global state would come from a store (cart, auth)
  // const { addToCart } = useCartStore();

  return (
    <div>
      <h1>{product?.name}</h1>
      <SizeSelector value={selectedSize} onChange={setSelectedSize} />
      <button onClick={() => setShowReviews(s => !s)}>
        {showReviews ? 'Hide' : 'Show'} Reviews
      </button>
    </div>
  );
}
```


## Resources

- [Managing State](https://react.dev/learn/managing-state) — Official React guide on state management

---

> 📘 *This lesson is part of the [React 19 & Patterns](https://stanza.dev/courses/react-modern-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*