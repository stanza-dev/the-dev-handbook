---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-designing-hooks"
---

# Designing Hook APIs

Create intuitive, flexible, and maintainable hooks.

## Return Value Patterns

### Single Value

```jsx
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  // ... effect
  return isOnline;
}

// Usage
const isOnline = useOnlineStatus();
```

### Tuple (useState-like)

```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  
  const setStoredValue = (newValue) => {
    setValue(newValue);
    localStorage.setItem(key, JSON.stringify(newValue));
  };
  
  return [value, setStoredValue]; // Like useState
}

// Usage - familiar pattern
const [name, setName] = useLocalStorage('name', '');
```

### Object (Multiple Values)

```jsx
function useFetch(url) {
  const [state, setState] = useState({
    data: null,
    loading: true,
    error: null,
  });
  
  // ... effect
  
  return state;
  // Or with actions:
  // return { ...state, refetch };
}

// Usage - destructure what you need
const { data, loading } = useFetch('/api/users');
```

## Parameter Patterns

### Required Parameters

```jsx
function useUser(userId) {
  // userId is required
  const { data } = useQuery(['user', userId], () => fetchUser(userId));
  return data;
}
```

### Optional Configuration

```jsx
function useFetch(url, options = {}) {
  const {
    enabled = true,
    refetchInterval = null,
    onSuccess = () => {},
    onError = () => {},
  } = options;
  
  // ... use options
}

// Usage
const { data } = useFetch('/api/users', {
  enabled: isReady,
  refetchInterval: 5000,
});
```

### Ref Parameters

```jsx
function useClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current?.contains(event.target)) {
        handler(event);
      }
    };
    
    document.addEventListener('mousedown', listener);
    return () => document.removeEventListener('mousedown', listener);
  }, [ref, handler]);
}

// Usage
function Dropdown() {
  const ref = useRef(null);
  useClickOutside(ref, () => setIsOpen(false));
  
  return <div ref={ref}>...</div>;
}
```

## Composition Patterns

### Hooks Using Hooks

```jsx
function useUser(userId) {
  return useFetch(`/api/users/${userId}`);
}

function useUserPosts(userId) {
  const { data: user } = useUser(userId);
  return useFetch(
    user ? `/api/users/${userId}/posts` : null,
    { enabled: !!user }
  );
}
```

### Hook Factory

```jsx
function createResourceHook(resourceName) {
  return function useResource(id) {
    return useFetch(`/api/${resourceName}/${id}`);
  };
}

const useUser = createResourceHook('users');
const usePost = createResourceHook('posts');
const useComment = createResourceHook('comments');
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*