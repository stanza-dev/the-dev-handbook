---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-effect-rune"
---

# The $effect Rune

Sometimes you need to run code **in response to** state changes — like logging, saving to localStorage, or updating the document title. This is what `$effect` is for.

## Basic Usage

```svelte
<script>
  let count = $state(0);

  $effect(() => {
    console.log(`The count is now ${count}`);
  });
</script>

<button onclick={() => count++}>Increment</button>
```

Every time `count` changes, the effect runs and logs the new value.

## When Does $effect Run?

1. **After the component mounts** (initial run)
2. **After any tracked dependency changes** (subsequent runs)
3. **After the DOM has been updated** (you can safely read DOM measurements)

## Automatic Dependency Tracking

Just like `$derived`, Svelte automatically tracks which reactive values you read inside the effect:

```svelte
<script>
  let name = $state('World');
  let greeting = $state('Hello');

  // Runs when name OR greeting changes
  $effect(() => {
    document.title = `${greeting}, ${name}!`;
  });
</script>
```

## Cleanup Function

Effects can return a **cleanup function** that runs before the effect re-runs or when the component unmounts:

```svelte
<script>
  let interval = $state(1000);

  $effect(() => {
    const id = setInterval(() => {
      console.log('Tick!');
    }, interval);

    // Cleanup: clear the interval
    return () => {
      clearInterval(id);
    };
  });
</script>
```

This is crucial for preventing memory leaks with timers, event listeners, or subscriptions.

## Real-World Example: LocalStorage

```svelte
<script>
  // Initialize from localStorage
  let theme = $state(
    typeof localStorage !== 'undefined' 
      ? localStorage.getItem('theme') || 'light'
      : 'light'
  );

  // Save to localStorage whenever theme changes
  $effect(() => {
    localStorage.setItem('theme', theme);
  });
</script>

<button onclick={() => theme = theme === 'light' ? 'dark' : 'light'}>
  Toggle Theme: {theme}
</button>
```

## Important: Effects vs Derived

- Use `$derived` when you need to **compute a value** from other state
- Use `$effect` when you need to **perform an action** (side effect) based on state

```svelte
// ✅ Good: Computing a value
let doubled = $derived(count * 2);

// ❌ Bad: Don't use effect to compute values
$effect(() => {
  doubled = count * 2;  // This is wrong!
});
```

## Resources

- [$effect Documentation](https://svelte.dev/docs/svelte/$effect) — Complete guide to side effects in Svelte 5.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*