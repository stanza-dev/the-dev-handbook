---
source_course: "react-performance"
source_lesson: "react-performance-memoization-pitfalls"
---

# Memoization Pitfalls

Common mistakes that break memoization.

## Pitfall 1: Unstable Dependencies

```jsx
// ❌ options is new every render
function Component() {
  const options = { sort: 'asc', limit: 10 };
  
  const data = useMemo(() => {
    return processData(options);
  }, [options]); // Always different!
}

// ✅ Move outside or memoize
const OPTIONS = { sort: 'asc', limit: 10 };

function Component() {
  const data = useMemo(() => {
    return processData(OPTIONS);
  }, []);
}

// ✅ Or memoize the options
function Component({ sort, limit }) {
  const options = useMemo(() => ({ sort, limit }), [sort, limit]);
  
  const data = useMemo(() => {
    return processData(options);
  }, [options]);
}
```

## Pitfall 2: Callback Using Stale Values

```jsx
// ❌ count is stale
function Counter() {
  const [count, setCount] = useState(0);
  
  const logCount = useCallback(() => {
    console.log(count); // Always logs initial value
  }, []); // count should be in deps
  
  return (
    <button onClick={logCount}>
      Count: {count}
    </button>
  );
}

// ✅ Include count in dependencies
const logCount = useCallback(() => {
  console.log(count);
}, [count]);

// ✅ Or use a ref for "escape hatch"
function Counter() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  countRef.current = count;
  
  const logCount = useCallback(() => {
    console.log(countRef.current); // Always current
  }, []);
}
```

## Pitfall 3: Premature Memoization

```jsx
// ❌ Over-memoizing simple values
function Component({ price, quantity }) {
  // Multiplication is cheap - no need to memoize
  const total = useMemo(() => price * quantity, [price, quantity]);
  
  // String concatenation is cheap
  const message = useMemo(() => `Total: $${total}`, [total]);
  
  return <span>{message}</span>;
}

// ✅ Just calculate inline
function Component({ price, quantity }) {
  const total = price * quantity;
  return <span>Total: ${total}</span>;
}
```

## Pitfall 4: Breaking memo with Spreading

```jsx
// ❌ Spreading props breaks memo
function Parent({ user, ...rest }) {
  return <MemoizedChild user={user} {...rest} />;
  // rest is new object every render!
}

// ✅ Be explicit about props
function Parent({ user, onSave, onCancel }) {
  return (
    <MemoizedChild 
      user={user} 
      onSave={onSave}
      onCancel={onCancel}
    />
  );
}
```

## Pitfall 5: Forgetting Children

```jsx
// ❌ children breaks memoization
function Parent() {
  return (
    <MemoizedWrapper>
      <Content /> {/* New JSX element every render */}
    </MemoizedWrapper>
  );
}

// ✅ Memoize children too
function Parent() {
  const content = useMemo(() => <Content />, []);
  return <MemoizedWrapper>{content}</MemoizedWrapper>;
}

// ✅ Or use render props
function Parent() {
  return (
    <MemoizedWrapper renderContent={() => <Content />} />
  );
}
```

## Debugging Memoization

```jsx
// Add logging to see when memo fails
const MemoizedComponent = memo(
  function Component(props) {
    console.log('Component rendered', props);
    return <div>{/* ... */}</div>;
  },
  (prevProps, nextProps) => {
    const areEqual = shallowEqual(prevProps, nextProps);
    if (!areEqual) {
      console.log('Props changed:', {
        prev: prevProps,
        next: nextProps,
      });
    }
    return areEqual;
  }
);
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*