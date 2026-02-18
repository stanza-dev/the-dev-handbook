---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-memoization"
---

# Memoization in React

React provides hooks to cache expensive calculations and function definitions between re-renders.

## useMemo: Caching Computed Values

`useMemo` returns a memoized value. It only recomputes when dependencies change.

```tsx
function ProductList({ products, filter }: Props) {
  // Only recalculates when products or filter changes
  const filteredProducts = useMemo(() => {
    console.log('Filtering products...');
    return products.filter((p) =>
      p.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [products, filter]);

  return (
    <ul>
      {filteredProducts.map((product) => (
        <ProductItem key={product.id} product={product} />
      ))}
    </ul>
  );
}
```

## useCallback: Caching Functions

`useCallback` returns a memoized callback. Essential when passing callbacks to optimized child components.

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Without useCallback, this creates a new function every render
  // causing ExpensiveChild to re-render unnecessarily
  const handleClick = useCallback(() => {
    console.log('Clicked!', count);
  }, [count]);

  return (
    <>
      <input value={name} onChange={(e) => setName(e.target.value)} />
      <ExpensiveChild onClick={handleClick} />
    </>
  );
}

const ExpensiveChild = memo(function ExpensiveChild({ onClick }: Props) {
  console.log('ExpensiveChild rendered');
  return <button onClick={onClick}>Click me</button>;
});
```

## When to Use (and When Not To)

### ✅ Use memoization when:

1. **Expensive calculations** that take >1ms
2. **Referential equality matters** (passing to memo'd children)
3. **Dependencies of other hooks** (useEffect, useMemo)

### ❌ Don't use when:

1. **Simple calculations** - memoization has overhead
2. **Primitive values** - they're compared by value anyway
3. **Every render** - if deps always change, it's wasted effort

## The React Compiler

React 19 introduces the React Compiler (formerly "React Forget") which automatically adds memoization. However, understanding these hooks remains important:

- Legacy codebases won't have the compiler
- Understanding helps debug performance issues
- Some cases still benefit from manual optimization

## Resources

- [useMemo API Reference](https://react.dev/reference/react/useMemo) — Official React documentation for useMemo hook
- [useCallback API Reference](https://react.dev/reference/react/useCallback) — Official React documentation for useCallback hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*