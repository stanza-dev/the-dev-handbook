---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-hooks"
---

# Hooks: Global Request/Response Handling

Hooks let you intercept requests and responses globally.

## Server Hooks (hooks.server.js)

```javascript
// src/hooks.server.js

// Runs for every request
export async function handle({ event, resolve }) {
  // Before route handlers
  const sessionId = event.cookies.get('session');
  
  if (sessionId) {
    const user = await getUserFromSession(sessionId);
    event.locals.user = user;  // Available in load functions
  }
  
  // Run route handlers
  const response = await resolve(event);
  
  // After route handlers (can modify response)
  return response;
}
```

## Authentication Example

```javascript
// src/hooks.server.js
import { redirect } from '@sveltejs/kit';

const protectedRoutes = ['/dashboard', '/settings', '/admin'];

export async function handle({ event, resolve }) {
  const session = event.cookies.get('session');
  
  if (session) {
    event.locals.user = await getUserFromSession(session);
  }
  
  // Protect routes
  const isProtected = protectedRoutes.some(route => 
    event.url.pathname.startsWith(route)
  );
  
  if (isProtected && !event.locals.user) {
    throw redirect(303, `/login?redirect=${event.url.pathname}`);
  }
  
  return resolve(event);
}
```

## Sequence Multiple Hooks

```javascript
import { sequence } from '@sveltejs/kit/hooks';

async function authentication({ event, resolve }) {
  // Auth logic
  return resolve(event);
}

async function logging({ event, resolve }) {
  console.log(`${event.request.method} ${event.url.pathname}`);
  const start = Date.now();
  const response = await resolve(event);
  console.log(`Completed in ${Date.now() - start}ms`);
  return response;
}

export const handle = sequence(authentication, logging);
```

## handleError Hook

```javascript
// src/hooks.server.js
export function handleError({ error, event }) {
  // Log to error tracking service
  console.error('Unhandled error:', error);
  
  // Return sanitized error for client
  return {
    message: 'Something went wrong',
    code: 'INTERNAL_ERROR'
  };
}
```

## Client Hooks (hooks.client.js)

```javascript
// src/hooks.client.js
export function handleError({ error }) {
  // Client-side error tracking
  errorTrackingService.capture(error);
}
```

📖 [Hooks documentation](https://svelte.dev/docs/kit/hooks)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*