---
source_course: "javascript"
source_lesson: "javascript-localstorage-json"
---

# Working with localStorage

## Introduction

localStorage provides persistent key-value storage in the browser. Since it only stores strings, JSON serialization is essential. Understanding the patterns and pitfalls helps build robust client-side data persistence.

## Key Concepts

**localStorage**: Browser API for persistent string key-value storage.

**sessionStorage**: Same API but cleared when browser session ends.

**Storage Limit**: Usually 5-10MB per origin.

## Deep Dive

### Basic Pattern

```javascript
// Storing objects
const user = { name: 'Alice', preferences: { theme: 'dark' } };
localStorage.setItem('user', JSON.stringify(user));

// Retrieving objects
const stored = localStorage.getItem('user');
const userData = stored ? JSON.parse(stored) : null;

// Removing
localStorage.removeItem('user');

// Clear all
localStorage.clear();
```

### Safe Storage Utility

```javascript
const storage = {
  get(key, fallback = null) {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : fallback;
    } catch {
      return fallback;
    }
  },
  
  set(key, value) {
    try {
      localStorage.setItem(key, JSON.stringify(value));
      return true;
    } catch (e) {
      // QuotaExceededError or SecurityError
      console.error('Storage failed:', e);
      return false;
    }
  },
  
  remove(key) {
    localStorage.removeItem(key);
  },
  
  has(key) {
    return localStorage.getItem(key) !== null;
  }
};

// Usage
storage.set('settings', { theme: 'dark', lang: 'en' });
const settings = storage.get('settings', { theme: 'light' });
```

### Handling Dates

```javascript
// Problem: Dates become strings
storage.set('event', { date: new Date() });
const event = storage.get('event');
typeof event.date;  // 'string'!

// Solution: Reviver or wrapper
const storageWithDates = {
  get(key, fallback = null) {
    const item = localStorage.getItem(key);
    if (!item) return fallback;
    return JSON.parse(item, (k, v) => {
      if (typeof v === 'string' && /^\d{4}-\d{2}-\d{2}T/.test(v)) {
        return new Date(v);
      }
      return v;
    });
  },
  // ... set as before
};
```

### Detecting Storage Changes

```javascript
// Listen for changes from OTHER tabs
window.addEventListener('storage', (e) => {
  console.log('Key:', e.key);
  console.log('Old:', e.oldValue);
  console.log('New:', e.newValue);
  console.log('URL:', e.url);
  
  // React to changes
  if (e.key === 'user') {
    updateUserDisplay(JSON.parse(e.newValue));
  }
});

// Note: doesn't fire for changes in same tab!
```

### Storage Quota Handling

```javascript
function safeStore(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
  } catch (e) {
    if (e.name === 'QuotaExceededError') {
      // Try to make space
      clearOldCache();
      // Retry
      localStorage.setItem(key, JSON.stringify(value));
    }
  }
}

// Check available space (approximate)
function getStorageUsage() {
  let total = 0;
  for (let i = 0; i < localStorage.length; i++) {
    const key = localStorage.key(i);
    total += localStorage.getItem(key).length;
  }
  return total;  // Characters (roughly bytes for ASCII)
}
```

## Common Pitfalls

1. **Forgetting to parse/stringify**: localStorage only stores strings.
2. **Not handling parse errors**: Corrupted data crashes the app.
3. **Assuming storage is available**: Private browsing may disable it.
4. **Storing sensitive data**: localStorage is not secure.

## Best Practices

- **Always wrap in try/catch**: Storage can fail.
- **Create a storage utility**: Handles JSON and errors consistently.
- **Use fallback values**: For when storage is empty or unavailable.
- **Don't store sensitive data**: Use secure cookies for auth tokens.

## Summary

localStorage stores strings, so JSON serialization is required. Create utility functions for safe get/set. Handle parse errors and storage quota limits. Listen to 'storage' event for cross-tab sync. Never store sensitive data in localStorage.

## Code Examples

**Basic Pattern**

```javascript
// Storing objects
const user = { name: 'Alice', preferences: { theme: 'dark' } };
localStorage.setItem('user', JSON.stringify(user));

// Retrieving objects
const stored = localStorage.getItem('user');
const userData = stored ? JSON.parse(stored) : null;

// Removing
localStorage.removeItem('user');

// Clear all
localStorage.clear();
```

**Safe Storage Utility**

```javascript
const storage = {
  get(key, fallback = null) {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : fallback;
    } catch {
      return fallback;
    }
  },
  
  set(key, value) {
    try {
      localStorage.setItem(key, JSON.stringify(value));
      return true;
    } catch (e) {
      // QuotaExceededError or SecurityError
      console.error('Storage failed:', e);
      return false;
```


## Resources

- [MDN: Window.localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage) — localStorage reference
- [MDN: StorageEvent](https://developer.mozilla.org/en-US/docs/Web/API/StorageEvent) — Storage event for cross-tab sync

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*