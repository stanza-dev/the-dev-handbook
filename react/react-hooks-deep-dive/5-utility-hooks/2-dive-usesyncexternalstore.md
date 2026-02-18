---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-usesyncexternalstore"
---

# useSyncExternalStore: Subscribing to External Stores

`useSyncExternalStore` lets you subscribe to an external store. It's primarily used by library authors but understanding it helps when working with external state.

## Basic Usage

```tsx
import { useSyncExternalStore } from 'react';

function useOnlineStatus() {
  const isOnline = useSyncExternalStore(
    subscribe,    // How to subscribe
    getSnapshot,  // How to get current value (client)
    getServerSnapshot // How to get value on server (optional)
  );
  return isOnline;
}

// Subscribe to online/offline events
function subscribe(callback: () => void) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

// Get current value
function getSnapshot() {
  return navigator.onLine;
}

// Server snapshot (for SSR)
function getServerSnapshot() {
  return true; // Assume online on server
}
```

## Real-World Example: Browser Storage

```tsx
function useLocalStorage<T>(key: string, initialValue: T) {
  const subscribe = useCallback((callback: () => void) => {
    window.addEventListener('storage', callback);
    return () => window.removeEventListener('storage', callback);
  }, []);

  const getSnapshot = useCallback(() => {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  }, [key, initialValue]);

  const getServerSnapshot = useCallback(() => initialValue, [initialValue]);

  const value = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);

  const setValue = useCallback((newValue: T) => {
    localStorage.setItem(key, JSON.stringify(newValue));
    // Dispatch event to notify other tabs/windows
    window.dispatchEvent(new StorageEvent('storage', { key }));
  }, [key]);

  return [value, setValue] as const;
}
```

## When to Use

- Subscribing to browser APIs (online status, media queries)
- Integrating with external state libraries
- Syncing with localStorage/sessionStorage
- Subscribing to third-party data sources

## Why Not Just useEffect?

`useSyncExternalStore` handles edge cases that are tricky with `useEffect`:

1. **Tearing**: Ensures consistent reads during concurrent rendering
2. **Hydration**: Properly handles server/client mismatches
3. **Subscription timing**: Subscribes synchronously to avoid missing updates

## Resources

- [useSyncExternalStore API Reference](https://react.dev/reference/react/useSyncExternalStore) — Official React documentation for useSyncExternalStore hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*