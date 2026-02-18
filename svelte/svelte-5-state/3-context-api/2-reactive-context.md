---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-reactive-context"
---

# Making Context Reactive

By itself, context just passes a static value. To make it reactive, combine it with `$state`.

## Non-Reactive Context (Static)

```svelte
<!-- Parent.svelte -->
<script>
  import { setContext } from 'svelte';
  
  // This value won't update in children if changed
  setContext('count', 0);
</script>
```

## Reactive Context with $state

```svelte
<!-- Parent.svelte -->
<script>
  import { setContext } from 'svelte';
  
  // Create reactive state
  let count = $state(0);
  
  // Pass an object with the reactive property
  setContext('counter', {
    get count() { return count; },
    increment: () => count++,
    decrement: () => count--
  });
</script>

<button onclick={() => count++}>Increment in parent: {count}</button>
<slot />
```

```svelte
<!-- Child.svelte -->
<script>
  import { getContext } from 'svelte';
  
  const counter = getContext('counter');
</script>

<!-- Reactively updates when parent changes! -->
<p>Count from context: {counter.count}</p>
<button onclick={counter.increment}>Increment from child</button>
```

## The Class Pattern (Recommended)

For complex state, use a class:

```javascript
// theme.svelte.js
import { setContext, getContext } from 'svelte';

const THEME_KEY = Symbol('theme');

export class ThemeState {
  mode = $state('light');
  
  get isDark() {
    return this.mode === 'dark';
  }
  
  toggle() {
    this.mode = this.mode === 'light' ? 'dark' : 'light';
  }
  
  setMode(mode) {
    this.mode = mode;
  }
}

export function setThemeContext() {
  const theme = new ThemeState();
  setContext(THEME_KEY, theme);
  return theme;
}

export function getThemeContext() {
  return getContext(THEME_KEY);
}
```

```svelte
<!-- App.svelte -->
<script>
  import { setThemeContext } from './theme.svelte.js';
  
  const theme = setThemeContext();
</script>

<div class:dark={theme.isDark}>
  <slot />
</div>
```

```svelte
<!-- DeepComponent.svelte -->
<script>
  import { getThemeContext } from './theme.svelte.js';
  
  const theme = getThemeContext();
</script>

<button onclick={() => theme.toggle()}>
  Current: {theme.mode}
</button>
```

📖 [Context documentation](https://svelte.dev/docs/svelte/context)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*