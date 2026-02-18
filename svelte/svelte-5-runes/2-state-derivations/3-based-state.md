---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-class-based-state"
---

# Encapsulating State in Classes

Classes provide a clean way to encapsulate related state and logic. Runes work naturally with class syntax.

## Basic Class Pattern

```js
// counter.svelte.js
export class Counter {
  count = $state(0);
  
  increment() {
    this.count += 1;
  }
  
  decrement() {
    this.count -= 1;
  }
  
  reset() {
    this.count = 0;
  }
}
```

Usage:

```svelte
<script>
  import { Counter } from './counter.svelte.js';
  
  const counter = new Counter();
</script>

<button onclick={() => counter.decrement()}>-</button>
<span>{counter.count}</span>
<button onclick={() => counter.increment()}>+</button>
```

## Derived Properties in Classes

```js
export class TodoList {
  items = $state([]);
  filter = $state('all');
  
  // Derived as a getter
  get filteredItems() {
    if (this.filter === 'all') return this.items;
    if (this.filter === 'active') return this.items.filter(t => !t.done);
    return this.items.filter(t => t.done);
  }
  
  get activeCount() {
    return this.items.filter(t => !t.done).length;
  }
  
  get allDone() {
    return this.items.length > 0 && this.items.every(t => t.done);
  }
  
  addTodo(text) {
    this.items.push({ id: Date.now(), text, done: false });
  }
  
  toggle(id) {
    const todo = this.items.find(t => t.id === id);
    if (todo) todo.done = !todo.done;
  }
  
  clearCompleted() {
    this.items = this.items.filter(t => !t.done);
  }
}
```

## Why Classes?

1. **Encapsulation**: State and methods live together
2. **Multiple Instances**: Each `new Counter()` is independent
3. **TypeScript Support**: Classes have excellent type inference
4. **Testability**: Easy to instantiate and test in isolation
5. **Familiarity**: Object-oriented pattern many developers know

## Class + Context = Dependency Injection

Combine classes with Svelte's Context API for SSR-safe shared state:

```js
// theme.svelte.js
import { setContext, getContext } from 'svelte';

const THEME_KEY = Symbol('theme');

export class Theme {
  mode = $state('light');
  
  toggle() {
    this.mode = this.mode === 'light' ? 'dark' : 'light';
  }
}

export function setTheme() {
  return setContext(THEME_KEY, new Theme());
}

export function getTheme() {
  return getContext(THEME_KEY);
}
```

In your root layout:

```svelte
<script>
  import { setTheme } from './theme.svelte.js';
  setTheme();  // Creates instance for this request/session
</script>
```

In any child:

```svelte
<script>
  import { getTheme } from './theme.svelte.js';
  const theme = getTheme();
</script>

<button onclick={() => theme.toggle()}>
  Current: {theme.mode}
</button>
```

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*