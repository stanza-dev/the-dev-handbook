---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-single-source-truth"
---

# Single Source of Truth

Avoid duplicate state to prevent synchronization bugs.

## The Problem: Duplicate State

```jsx
// ❌ Bad: Two sources of truth
function UserProfile({ user }) {
  const [localUser, setLocalUser] = useState(user);
  
  // Now we have user AND localUser
  // Which is correct?
  // They can get out of sync!
}
```

## Solution: Single Owner

```jsx
// ✅ Props are the single source
function UserProfile({ user, onUpdate }) {
  // Use user directly
  // Call onUpdate to change it
  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={() => onUpdate({ ...user, name: 'New Name' })}>
        Update
      </button>
    </div>
  );
}
```

## Controlled vs Uncontrolled

```jsx
// Controlled: Parent owns the state
function ControlledInput({ value, onChange }) {
  return <input value={value} onChange={(e) => onChange(e.target.value)} />;
}

// Usage
function Parent() {
  const [name, setName] = useState('');
  return <ControlledInput value={name} onChange={setName} />;
}

// Uncontrolled: Component owns its state
function UncontrolledInput({ defaultValue, onChange }) {
  const [value, setValue] = useState(defaultValue);
  
  const handleChange = (e) => {
    setValue(e.target.value);
    onChange?.(e.target.value);
  };
  
  return <input value={value} onChange={handleChange} />;
}
```

## Avoid Syncing State

```jsx
// ❌ Bad: Syncing state with useEffect
function UserForm({ user }) {
  const [name, setName] = useState(user.name);
  const [email, setEmail] = useState(user.email);
  
  // This is a red flag!
  useEffect(() => {
    setName(user.name);
    setEmail(user.email);
  }, [user]);
  
  return /* ... */;
}

// ✅ Good: Key to reset state
function UserForm({ user }) {
  const [name, setName] = useState(user.name);
  const [email, setEmail] = useState(user.email);
  
  return /* ... */;
}

// Parent uses key to reset
function Parent({ userId }) {
  const user = useUser(userId);
  return <UserForm key={user.id} user={user} />;
}
```

## Normalizing State

```jsx
// ❌ Bad: Nested, duplicated data
const [posts, setPosts] = useState([
  {
    id: 1,
    title: 'Hello',
    author: { id: 1, name: 'Alice' }, // Duplicated!
  },
  {
    id: 2,
    title: 'World',
    author: { id: 1, name: 'Alice' }, // Same author, duplicated
  },
]);

// ✅ Good: Normalized state
const [state, setState] = useState({
  posts: {
    1: { id: 1, title: 'Hello', authorId: 1 },
    2: { id: 2, title: 'World', authorId: 1 },
  },
  users: {
    1: { id: 1, name: 'Alice' },
  },
});
```

## IDs Instead of Objects

```jsx
// ❌ Bad: Storing full objects
const [selectedUser, setSelectedUser] = useState(users[0]);

// ✅ Good: Store ID, derive the rest
const [selectedUserId, setSelectedUserId] = useState(users[0]?.id);
const selectedUser = users.find(u => u.id === selectedUserId);
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*