---
source_course: "react-performance"
source_lesson: "react-performance-react-memo"
---

# React.memo Deep Dive

`memo` prevents re-renders when props haven't changed.

## Basic Usage

```jsx
import { memo } from 'react';

const ExpensiveList = memo(function ExpensiveList({ items }) {
  console.log('ExpensiveList rendered');
  
  return (
    <ul>
      {items.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
});
```

## How memo Works

```jsx
// Pseudo-implementation
function memo(Component) {
  let prevProps = null;
  let prevResult = null;
  
  return function MemoizedComponent(props) {
    if (prevProps !== null && shallowEqual(prevProps, props)) {
      return prevResult; // Skip render, return cached result
    }
    
    prevProps = props;
    prevResult = <Component {...props} />;
    return prevResult;
  };
}
```

## Shallow Comparison

```jsx
// ✅ These prevent re-render (primitives)
<MemoizedChild name="John" age={30} />

// ❌ These cause re-render (new references)
<MemoizedChild
  user={{ name: 'John' }}     // New object
  items={['a', 'b']}          // New array
  onClick={() => {}}          // New function
/>
```

## Custom Comparison Function

```jsx
const UserCard = memo(
  function UserCard({ user, onClick }) {
    return (
      <div onClick={onClick}>
        <h2>{user.name}</h2>
        <p>{user.email}</p>
      </div>
    );
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    // Return false if props are different (re-render)
    return (
      prevProps.user.id === nextProps.user.id &&
      prevProps.user.name === nextProps.user.name
    );
  }
);
```

## When to Use memo

### ✅ Good Candidates

```jsx
// 1. Pure presentational components
const Avatar = memo(function Avatar({ src, alt }) {
  return <img src={src} alt={alt} className="avatar" />;
});

// 2. Components that render often with same props
const TableRow = memo(function TableRow({ data }) {
  return <tr>{/* ... */}</tr>;
});

// 3. Components with expensive render logic
const Chart = memo(function Chart({ data }) {
  const processedData = expensiveComputation(data);
  return <svg>{/* render chart */}</svg>;
});
```

### ❌ Avoid memo

```jsx
// 1. Components that almost always receive new props
// 2. Very simple/cheap components
// 3. Components that use children prop
const Wrapper = memo(function Wrapper({ children }) {
  // children is almost always a new reference!
  return <div className="wrapper">{children}</div>;
});
```

## memo with Children

```jsx
// ❌ Doesn't help - children is new every time
<MemoizedLayout>
  <Content />
</MemoizedLayout>

// ✅ Better - lift children up
function Parent() {
  const content = useMemo(() => <Content />, []);
  return <MemoizedLayout>{content}</MemoizedLayout>;
}

// ✅ Or use composition
function Parent() {
  return (
    <MemoizedLayout>
      <MemoizedContent />
    </MemoizedLayout>
  );
}
```

---

> 📘 *This lesson is part of the [React Performance Deep Dive](https://stanza.dev/courses/react-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*