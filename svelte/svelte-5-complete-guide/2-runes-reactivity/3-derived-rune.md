---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-derived-rune"
---

# The $derived Rune

Often you need values that are **computed from other state**. Instead of manually updating these values, use `$derived` to let Svelte handle it automatically.

## Basic Usage

```svelte
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);
  let quadrupled = $derived(doubled * 2);
</script>

<p>Count: {count}</p>
<p>Doubled: {doubled}</p>
<p>Quadrupled: {quadrupled}</p>
<button onclick={() => count++}>Increment</button>
```

When `count` changes:
1. `doubled` automatically recalculates to `count * 2`
2. `quadrupled` automatically recalculates to `doubled * 2`

You never manually update derived values — they stay in sync automatically!

## Why Not Just Use a Function?

You might think: "Why not just write a function?"

```svelte
<!-- Don't do this for derived values -->
<p>Doubled: {count * 2}</p>
<p>Doubled: {count * 2}</p>
<p>Doubled: {count * 2}</p>
```

The problem? The expression `count * 2` runs **every time it's used**. With `$derived`:

```svelte
let doubled = $derived(count * 2);

<p>Doubled: {doubled}</p>
<p>Doubled: {doubled}</p>
<p>Doubled: {doubled}</p>
```

The calculation runs **once** and is cached. Much more efficient!

## Complex Derivations with $derived.by

For complex calculations that need multiple statements, use `$derived.by()`:

```svelte
<script>
  let items = $state([10, 20, 30, 40, 50]);
  let threshold = $state(25);

  // Complex logic with multiple steps
  let stats = $derived.by(() => {
    const filtered = items.filter(item => item > threshold);
    const sum = filtered.reduce((a, b) => a + b, 0);
    const average = filtered.length > 0 ? sum / filtered.length : 0;
    
    return {
      count: filtered.length,
      sum,
      average: average.toFixed(2)
    };
  });
</script>

<p>Items above {threshold}: {stats.count}</p>
<p>Sum: {stats.sum}</p>
<p>Average: {stats.average}</p>
```

## Dependency Tracking

Svelte automatically tracks which reactive values are used inside `$derived`. It only recalculates when those specific dependencies change:

```svelte
<script>
  let firstName = $state('John');
  let lastName = $state('Doe');
  let age = $state(30);

  // Only recalculates when firstName or lastName changes
  // Does NOT recalculate when age changes
  let fullName = $derived(`${firstName} ${lastName}`);
</script>
```

## Resources

- [$derived Documentation](https://svelte.dev/docs/svelte/$derived) — Official docs for $derived and $derived.by.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*