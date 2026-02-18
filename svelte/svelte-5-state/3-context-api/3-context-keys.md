---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-context-keys"
---

# Choosing Context Keys

The key you use for `setContext` and `getContext` matters. Let's look at best practices.

## String Keys (Simple but Risky)

```javascript
// Works, but could collide with other libraries
setContext('theme', themeValue);
setContext('user', userValue);
```

**Problem:** What if a library you use also sets `'theme'` context?

## Symbol Keys (Recommended)

```javascript
// theme.svelte.js
const THEME_KEY = Symbol('theme');

export function setThemeContext(theme) {
  setContext(THEME_KEY, theme);
}

export function getThemeContext() {
  return getContext(THEME_KEY);
}
```

**Benefits:**
- Symbols are unique - no collisions possible
- Keys are private - only code with the Symbol can access
- Better encapsulation

## The Factory Pattern

Combine Symbol keys with helper functions:

```javascript
// auth.svelte.js
import { setContext, getContext } from 'svelte';

const AUTH_KEY = Symbol('auth');

export class AuthState {
  user = $state(null);
  isLoading = $state(false);
  
  get isAuthenticated() {
    return this.user !== null;
  }
  
  async login(credentials) {
    this.isLoading = true;
    try {
      this.user = await api.login(credentials);
    } finally {
      this.isLoading = false;
    }
  }
  
  logout() {
    this.user = null;
  }
}

export function setAuthContext() {
  const auth = new AuthState();
  setContext(AUTH_KEY, auth);
  return auth;
}

export function getAuthContext() {
  const auth = getContext(AUTH_KEY);
  if (!auth) {
    throw new Error('Auth context not found. Did you forget to call setAuthContext?');
  }
  return auth;
}
```

## Multiple Context Values

You can set multiple context values:

```svelte
<script>
  import { setThemeContext } from './theme.svelte.js';
  import { setAuthContext } from './auth.svelte.js';
  import { setCartContext } from './cart.svelte.js';
  
  // Each uses its own Symbol key - no conflicts
  setThemeContext();
  setAuthContext();
  setCartContext();
</script>
```

## hasContext for Optional Features

```javascript
import { hasContext, getContext } from 'svelte';

function maybeGetTheme() {
  if (hasContext(THEME_KEY)) {
    return getContext(THEME_KEY);
  }
  // Return default if context not available
  return { mode: 'light' };
}
```

📖 [Context documentation](https://svelte.dev/docs/svelte/context)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*