---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-state-location"
---

# Where Should State Live?

Choosing the right location for state prevents complexity.

## The Principle: Lift Only When Needed

```jsx
// ❌ Don't lift unnecessarily
// Parent doesn't need to know about hover state
function Parent() {
  const [isButtonHovered, setIsButtonHovered] = useState(false);
  return <Button isHovered={isButtonHovered} onHover={setIsButtonHovered} />;
}

// ✅ Keep state local when possible
function Button() {
  const [isHovered, setIsHovered] = useState(false);
  return (
    <button
      onMouseEnter={() => setIsHovered(true)}
      onMouseLeave={() => setIsHovered(false)}
    >
      Click me
    </button>
  );
}
```

## When to Lift State

### 1. Siblings Need to Share

```jsx
function ProductPage() {
  const [selectedSize, setSelectedSize] = useState('M');
  
  return (
    <>
      {/* Both need selectedSize */}
      <SizeSelector
        selected={selectedSize}
        onChange={setSelectedSize}
      />
      <AddToCartButton size={selectedSize} />
    </>
  );
}
```

### 2. Parent Needs to React

```jsx
function Form() {
  const [isValid, setIsValid] = useState(false);
  
  return (
    <>
      <EmailInput onValidChange={setIsValid} />
      <SubmitButton disabled={!isValid} />
    </>
  );
}
```

## Colocation Pattern

Keep state close to where it's used:

```jsx
// ❌ State too far from usage
function App() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <Layout>
      <Header>
        <SearchBar value={searchQuery} onChange={setSearchQuery} />
      </Header>
      <Main>
        <SearchResults query={searchQuery} />
      </Main>
    </Layout>
  );
}

// ✅ State colocated with components that need it
function SearchSection() {
  const [searchQuery, setSearchQuery] = useState('');
  return (
    <>
      <SearchBar value={searchQuery} onChange={setSearchQuery} />
      <SearchResults query={searchQuery} />
    </>
  );
}
```

## Derived State vs Stored State

```jsx
// ❌ Redundant state
function ProductList({ products }) {
  const [filteredProducts, setFilteredProducts] = useState(products);
  const [filter, setFilter] = useState('');

  useEffect(() => {
    setFilteredProducts(
      products.filter(p => p.name.includes(filter))
    );
  }, [products, filter]);

  return /* ... */;
}

// ✅ Derive instead of store
function ProductList({ products }) {
  const [filter, setFilter] = useState('');
  
  // Computed during render
  const filteredProducts = products.filter(
    p => p.name.includes(filter)
  );

  return /* ... */;
}
```

## State Location Checklist

1. **Can it be derived?** → Don't store, compute
2. **Only one component needs it?** → Local state
3. **Parent and children need it?** → Lift to parent
4. **Distant components need it?** → Context or store
5. **From server?** → Server state library
6. **In URL?** → Router state

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*