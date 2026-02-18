---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-state-rune"
---

# The $state Rune

`$state` is the foundation of reactivity in Svelte 5. It creates a reactive variable that, when changed, automatically updates any part of the UI that uses it.

## Basic Usage

```svelte
<script>
  let count = $state(0);

  function increment() {
    count += 1;  // This triggers a UI update!
  }
</script>

<button onclick={increment}>
  Clicked {count} times
</button>
```

When you click the button, `count` changes, and Svelte automatically updates the text in the button. No manual DOM manipulation needed!

## How $state Works

Under the hood, `$state` creates a **reactive proxy**. This means:

1. Svelte tracks which parts of your UI use this value
2. When the value changes, only those specific parts update
3. Updates are surgical — Svelte doesn't re-render your entire component

## Deep Reactivity

`$state` is **deeply reactive** for objects and arrays. You can mutate nested properties and Svelte will detect the changes:

```svelte
<script>
  let user = $state({
    name: 'Alice',
    settings: {
      theme: 'dark',
      notifications: true
    }
  });

  function toggleTheme() {
    // This works! Deep mutation is tracked
    user.settings.theme = user.settings.theme === 'dark' ? 'light' : 'dark';
  }
</script>

<p>Theme: {user.settings.theme}</p>
<button onclick={toggleTheme}>Toggle Theme</button>
```

## Arrays Are Reactive Too

You can use array methods like `push`, `pop`, `splice`, and they'll trigger updates:

```svelte
<script>
  let todos = $state(['Learn Svelte', 'Build app']);

  function addTodo() {
    todos.push('New todo');  // This triggers an update!
  }
  
  function removeLast() {
    todos.pop();  // This also triggers an update!
  }
</script>

<ul>
  {#each todos as todo}
    <li>{todo}</li>
  {/each}
</ul>
```

## Primitive vs Object State

For simple values (numbers, strings, booleans), the reactive variable holds the value directly:

```svelte
let count = $state(0);
count += 1;  // Direct assignment
```

For objects and arrays, you're working with a proxy that wraps your data:

```svelte
let user = $state({ name: 'Alice' });
user.name = 'Bob';  // Mutate properties directly
```

**Tip**: If you ever need the "raw" (non-reactive) version of state, you can use `$state.snapshot(value)` to get a plain object copy.

## Resources

- [$state Documentation](https://svelte.dev/docs/svelte/$state) — Complete guide to the $state rune.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*