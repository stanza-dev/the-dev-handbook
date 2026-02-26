---
source_course: "javascript-async-patterns"
source_lesson: "javascript-async-patterns-service-workers"
---

# Service Workers Overview

## Introduction

Service Workers are a special type of worker that act as a proxy between your app and the network. They enable offline functionality, push notifications, and background sync. They're the foundation of Progressive Web Apps (PWAs).

## Key Concepts

**Service Worker**: A script that runs separately from your page, intercepting network requests.

**Lifecycle**: install → activate → fetch (different from regular workers).

**Cache API**: Storage for request/response pairs.

## Deep Dive

### Registration

```javascript
// main.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('SW registered:', reg.scope))
    .catch(err => console.error('SW failed:', err));
}
```

### Basic Service Worker

```javascript
// sw.js
const CACHE_NAME = 'v1';
const ASSETS = ['/', '/index.html', '/styles.css', '/app.js'];

// Install - cache assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(ASSETS))
  );
});

// Activate - clean old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(keys => 
      Promise.all(
        keys.filter(key => key !== CACHE_NAME)
          .map(key => caches.delete(key))
      )
    )
  );
});

// Fetch - serve from cache, fallback to network
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request)
      .then(cached => cached || fetch(event.request))
  );
});
```

### Caching Strategies

```javascript
// Cache First (good for static assets)
async function cacheFirst(request) {
  const cached = await caches.match(request);
  return cached || fetch(request);
}

// Network First (good for API calls)
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    const cache = await caches.open(CACHE_NAME);
    cache.put(request, response.clone());
    return response;
  } catch {
    return caches.match(request);
  }
}

// Stale While Revalidate
async function staleWhileRevalidate(request) {
  const cache = await caches.open(CACHE_NAME);
  const cached = await cache.match(request);
  
  const fetchPromise = fetch(request).then(response => {
    cache.put(request, response.clone());
    return response;
  });
  
  return cached || fetchPromise;
}
```

## Common Pitfalls

1. **HTTPS required**: Service workers only work on HTTPS (except localhost).
2. **Scope matters**: SW only controls pages in its scope directory.
3. **Updates need refresh**: New SW waits until all old tabs close.

## Summary

Service Workers enable offline support and network interception. They have a lifecycle: install → activate → fetch. Cache API stores responses. Different strategies suit different content types. HTTPS is required.

## Code Examples

**Registration**

```javascript
// main.js
if ('serviceWorker' in navigator) {
  navigator.serviceWorker.register('/sw.js')
    .then(reg => console.log('SW registered:', reg.scope))
    .catch(err => console.error('SW failed:', err));
}
```

**Basic Service Worker**

```javascript
// sw.js
const CACHE_NAME = 'v1';
const ASSETS = ['/', '/index.html', '/styles.css', '/app.js'];

// Install - cache assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then(cache => cache.addAll(ASSETS))
  );
});

// Activate - clean old caches
self.addEventListener('activate', (event) => {
  event.waitUntil(
    caches.keys().then(keys => 
      Promise.all(
        keys.filter(key => key !== CACHE_NAME)
```


## Resources

- [MDN: Service Worker API](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API) — Service Worker API reference
- [MDN: Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers) — Service Worker guide

---

> 📘 *This lesson is part of the [Async JavaScript: Promises & Patterns](https://stanza.dev/courses/javascript-async-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*