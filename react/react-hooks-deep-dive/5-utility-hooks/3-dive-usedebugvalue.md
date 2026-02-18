---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-usedebugvalue"
---

# useDebugValue: Debugging Custom Hooks

`useDebugValue` adds a label to custom hooks in React DevTools, making debugging easier.

## Basic Usage

```tsx
import { useDebugValue, useState, useEffect } from 'react';

function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);

  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  // Shows "OnlineStatus: Online" or "OnlineStatus: Offline" in DevTools
  useDebugValue(isOnline ? 'Online' : 'Offline');

  return isOnline;
}
```

## Deferred Formatting

For expensive formatting, pass a format function as the second argument:

```tsx
function useUser(userId: string) {
  const user = useFetchUser(userId);

  // Format function only called when DevTools is open
  useDebugValue(user, (user) => {
    if (!user) return 'Loading...';
    return `${user.name} (${user.role})`;
  });

  return user;
}
```

## Best Practices

✅ **Use for:**
- Custom hooks in shared libraries
- Complex hooks where the internal state isn't obvious
- Hooks that would benefit from a human-readable label

❌ **Don't use for:**
- Simple hooks where the value is obvious
- Every custom hook (adds noise)
- Production performance-critical code (though impact is minimal)

## Example: Date Formatting Hook

```tsx
function useFormattedDate(date: Date, locale = 'en-US') {
  const formatted = useMemo(() => {
    return new Intl.DateTimeFormat(locale, {
      dateStyle: 'full',
      timeStyle: 'short'
    }).format(date);
  }, [date, locale]);

  useDebugValue(date, (d) => d.toISOString());

  return formatted;
}
```

In DevTools, you'll see: `FormattedDate: "2026-01-14T10:30:00.000Z"`

## Resources

- [useDebugValue API Reference](https://react.dev/reference/react/useDebugValue) — Official React documentation for useDebugValue hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*