---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-converting-state"
---

# Reactive Variables with $state

In Svelte 4, top-level `let` declarations were automatically reactive. In Svelte 5, you explicitly mark them.

## Simple Values

```svelte
<!-- Svelte 4 -->
<script>
  let count = 0;
  let name = 'world';
</script>

<!-- Svelte 5 -->
<script>
  let count = $state(0);
  let name = $state('world');
</script>
```

## Objects and Arrays

```svelte
<!-- Svelte 4 -->
<script>
  let user = { name: 'Alice', age: 30 };
  let items = ['a', 'b', 'c'];
</script>

<!-- Svelte 5 -->
<script>
  let user = $state({ name: 'Alice', age: 30 });
  let items = $state(['a', 'b', 'c']);
</script>
```

## Bonus: Deep Reactivity

Svelte 5's `$state` is deeply reactive by default:

```svelte
<script>
  let user = $state({ name: 'Alice', settings: { theme: 'dark' } });
</script>

<!-- This updates automatically in Svelte 5! -->
<button onclick={() => user.settings.theme = 'light'}>
  Toggle
</button>
```

In Svelte 4, you had to reassign the whole object.

## Non-Reactive Variables

Some variables don't need to be reactive:

```svelte
<script>
  // Constants - never change
  const API_URL = 'https://api.example.com';
  
  // References - just need to exist
  let element; // For bind:this
  
  // Internal tracking - not displayed
  let cache = new Map(); // Use $state only if displayed
</script>
```

## Class Fields

```svelte
<script>
  class Counter {
    count = $state(0);
    
    increment() {
      this.count++;
    }
  }
  
  const counter = new Counter();
</script>

<button onclick={() => counter.increment()}>
  {counter.count}
</button>
```

📖 [$state documentation](https://svelte.dev/docs/svelte/$state)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*