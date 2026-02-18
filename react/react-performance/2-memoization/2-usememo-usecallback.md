---
source_course: "react-performance"
source_lesson: "react-performance-usememo-usecallback"
---

# useMemo and useCallback

Memoize values and callbacks to maintain referential equality.

## useMemo: Memoize Values

```jsx
function ProductList({ products, filter }) {
  // Expensive filtering - only recompute when dependencies change
  const filteredProducts = useMemo(() => {
    console.log('Filtering products...');
    return products.filter(p => 
      p.category === filter.category &&
      p.price >= filter.minPrice &&
      p.price <= filter.maxPrice
    );
  }, [products, filter]);
  
  return (
    <ul>
      {filteredProducts.map(p => (
        <ProductItem key={p.id} product={p} />
      ))}
    </ul>
  );
}
```

## useCallback: Memoize Functions

```jsx
function SearchForm({ onSearch }) {
  const [query, setQuery] = useState('');
  
  // Without useCallback: new function every render
  // Child components receiving this would re-render
  const handleSubmit = useCallback((e) => {
    e.preventDefault();
    onSearch(query);
  }, [query, onSearch]);
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={query} onChange={e => setQuery(e.target.value)} />
      <SearchButton onClick={handleSubmit} />
    </form>
  );
}

const SearchButton = memo(function SearchButton({ onClick }) {
  console.log('SearchButton rendered');
  return <button onClick={onClick}>Search</button>;
});
```

## When NOT to Use

```jsx
// ❌ Over-optimization: Simple calculation
const doubled = useMemo(() => count * 2, [count]);
// ✅ Just calculate it
const doubled = count * 2;

// ❌ Over-optimization: No memoized children
function Parent() {
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  // Child isn't memoized, so useCallback is pointless
  return <Child onClick={handleClick} />;
}

// ✅ useCallback helps when Child is memoized
const Child = memo(function Child({ onClick }) {
  return <button onClick={onClick}>Click</button>;
});
```

## Dependency Arrays

```jsx
function Component({ userId, options }) {
  // ❌ Missing dependency
  const fetchUser = useCallback(() => {
    return api.getUser(userId); // userId used but not in deps
  }, []); // Bug: stale userId
  
  // ✅ Correct dependencies
  const fetchUser = useCallback(() => {
    return api.getUser(userId);
  }, [userId]);
  
  // ❌ Object in dependencies - new every render
  const processOptions = useMemo(() => {
    return transformOptions(options);
  }, [options]); // options is new object each render!
  
  // ✅ Use specific properties
  const processOptions = useMemo(() => {
    return transformOptions(options);
  }, [options.sort, options.filter, options.page]);
}
```

## The useMemo/useCallback Pattern

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);
  
  // Memoize data transformation
  const processedItems = useMemo(() => {
    return items.map(item => ({
      ...item,
      displayName: `${item.firstName} ${item.lastName}`,
    }));
  }, [items]);
  
  // Memoize callback
  const handleItemClick = useCallback((id) => {
    console.log('Clicked:', id);
  }, []);
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      {/* MemoizedList won't re-render when count changes */}
      <MemoizedList items={processedItems} onItemClick={handleItemClick} />
    </>
  );
}
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*