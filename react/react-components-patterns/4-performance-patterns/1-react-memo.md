---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-react-memo"
---

# React.memo: Skipping Unnecessary Renders

`memo` lets you skip re-rendering a component when its props haven't changed.

## Basic Usage

```tsx
import { memo } from 'react';

type UserCardProps = {
  user: User;
  onSelect: (id: string) => void;
};

const UserCard = memo(function UserCard({ user, onSelect }: UserCardProps) {
  console.log('UserCard rendered:', user.name);
  
  return (
    <div className="user-card" onClick={() => onSelect(user.id)}>
      <img src={user.avatar} alt={user.name} />
      <h3>{user.name}</h3>
    </div>
  );
});
```

## When memo Helps

```tsx
function UserList() {
  const [users, setUsers] = useState<User[]>([]);
  const [filter, setFilter] = useState('');
  
  // Without memo: ALL UserCards re-render when filter changes
  // With memo: Only UserCards whose props changed re-render
  
  return (
    <div>
      <input 
        value={filter} 
        onChange={e => setFilter(e.target.value)} 
      />
      {users.map(user => (
        <UserCard key={user.id} user={user} onSelect={handleSelect} />
      ))}
    </div>
  );
}
```

## The Callback Problem

`memo` uses shallow comparison. New function references break memoization:

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ New function every render - memo won't help
  const handleClick = () => console.log('clicked');
  
  return <MemoizedChild onClick={handleClick} />;
}

// ✅ Fix with useCallback
function Parent() {
  const [count, setCount] = useState(0);
  
  const handleClick = useCallback(() => {
    console.log('clicked');
  }, []);
  
  return <MemoizedChild onClick={handleClick} />;
}
```

## Custom Comparison

Provide a custom comparison function:

```tsx
const UserCard = memo(
  function UserCard({ user, onSelect }: UserCardProps) {
    return (/* ... */);
  },
  (prevProps, nextProps) => {
    // Return true if props are equal (skip re-render)
    // Return false if props are different (re-render)
    return prevProps.user.id === nextProps.user.id &&
           prevProps.user.name === nextProps.user.name;
  }
);
```

## When NOT to Use memo

```tsx
// ❌ Props change every render anyway
const Child = memo(({ data }) => <div>{data}</div>);
<Child data={{ value: count }} /> // New object every render!

// ❌ Component is cheap to render
const Label = memo(({ text }) => <span>{text}</span>);
// memo overhead > render cost

// ❌ Component always receives new props
const Timer = memo(({ time }) => <span>{time}</span>);
// time changes every second - memo never helps
```

## React Compiler Note

The React Compiler (React 19+) automatically adds memoization where beneficial. However, understanding `memo` is still important for:

- Legacy codebases
- Cases where the compiler can't optimize
- Understanding performance characteristics

## Resources

- [memo API Reference](https://react.dev/reference/react/memo) — Official React documentation for memo

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*