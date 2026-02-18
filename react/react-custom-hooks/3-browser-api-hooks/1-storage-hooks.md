---
source_course: "react-custom-hooks"
source_lesson: "react-custom-hooks-storage-hooks"
---

# useLocalStorage and useSessionStorage

Persist state in browser storage.

## useLocalStorage

```jsx
function useLocalStorage(key, initialValue) {
  // State to store our value
  const [storedValue, setStoredValue] = useState(() => {
    if (typeof window === 'undefined') {
      return initialValue;
    }
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Return a wrapped version of useState's setter function
  const setValue = useCallback((value) => {
    try {
      // Allow value to be a function
      const valueToStore = value instanceof Function 
        ? value(storedValue) 
        : value;
      
      setStoredValue(valueToStore);
      
      if (typeof window !== 'undefined') {
        window.localStorage.setItem(key, JSON.stringify(valueToStore));
      }
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  const removeValue = useCallback(() => {
    try {
      setStoredValue(initialValue);
      if (typeof window !== 'undefined') {
        window.localStorage.removeItem(key);
      }
    } catch (error) {
      console.error(error);
    }
  }, [key, initialValue]);
  
  return [storedValue, setValue, removeValue];
}

// Usage
function App() {
  const [theme, setTheme, clearTheme] = useLocalStorage('theme', 'light');
  
  return (
    <div className={theme}>
      <button onClick={() => setTheme(t => t === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
      <button onClick={clearTheme}>Reset</button>
    </div>
  );
}
```

## With Cross-Tab Sync

```jsx
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  // Sync across tabs
  useEffect(() => {
    const handleStorageChange = (e) => {
      if (e.key === key && e.newValue) {
        setStoredValue(JSON.parse(e.newValue));
      }
    };
    
    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);
  
  const setValue = useCallback((value) => {
    const valueToStore = value instanceof Function 
      ? value(storedValue) 
      : value;
    
    setStoredValue(valueToStore);
    localStorage.setItem(key, JSON.stringify(valueToStore));
  }, [key, storedValue]);
  
  return [storedValue, setValue];
}
```

## useSessionStorage

```jsx
function useSessionStorage(key, initialValue) {
  // Same implementation, different storage
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = sessionStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setValue = useCallback((value) => {
    const valueToStore = value instanceof Function 
      ? value(storedValue) 
      : value;
    setStoredValue(valueToStore);
    sessionStorage.setItem(key, JSON.stringify(valueToStore));
  }, [key, storedValue]);
  
  return [storedValue, setValue];
}
```

## Generic Storage Hook

```tsx
function useStorage<T>(
  storage: Storage,
  key: string,
  initialValue: T
): [T, (value: T | ((prev: T) => T)) => void] {
  const [value, setValue] = useState<T>(() => {
    try {
      const item = storage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setStorageValue = useCallback((newValue: T | ((prev: T) => T)) => {
    setValue(prev => {
      const resolved = newValue instanceof Function ? newValue(prev) : newValue;
      storage.setItem(key, JSON.stringify(resolved));
      return resolved;
    });
  }, [key, storage]);
  
  return [value, setStorageValue];
}

// Create specific hooks
const useLocalStorage = <T,>(key: string, initial: T) =>
  useStorage(localStorage, key, initial);

const useSessionStorage = <T,>(key: string, initial: T) =>
  useStorage(sessionStorage, key, initial);
```

---

> 📘 *This lesson is part of the [Building Custom Hooks](https://stanza.dev/courses/react-custom-hooks) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*