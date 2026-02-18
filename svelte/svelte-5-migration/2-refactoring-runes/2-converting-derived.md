---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-converting-derived"
---

# Reactive Statements to $derived

Svelte 4's `$:` label for derived values becomes `$derived` in Svelte 5.

## Simple Derivations

```svelte
<!-- Svelte 4 -->
<script>
  let count = 0;
  $: doubled = count * 2;
  $: quadrupled = doubled * 2;
</script>

<!-- Svelte 5 -->
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);
  let quadrupled = $derived(doubled * 2);
</script>
```

## Complex Derivations

For multi-line computations, use `$derived.by()`:

```svelte
<!-- Svelte 4 -->
<script>
  let items = [];
  let filter = '';
  
  $: filteredItems = items
    .filter(item => item.name.includes(filter))
    .sort((a, b) => a.name.localeCompare(b.name));
</script>

<!-- Svelte 5 -->
<script>
  let items = $state([]);
  let filter = $state('');
  
  let filteredItems = $derived.by(() => {
    return items
      .filter(item => item.name.includes(filter))
      .sort((a, b) => a.name.localeCompare(b.name));
  });
</script>
```

## Multiple Derived Values

```svelte
<!-- Svelte 4 -->
<script>
  let price = 100;
  let quantity = 2;
  
  $: subtotal = price * quantity;
  $: tax = subtotal * 0.1;
  $: total = subtotal + tax;
</script>

<!-- Svelte 5 -->
<script>
  let price = $state(100);
  let quantity = $state(2);
  
  let subtotal = $derived(price * quantity);
  let tax = $derived(subtotal * 0.1);
  let total = $derived(subtotal + tax);
</script>
```

## Important Difference

`$derived` values are truly derived - you can't assign to them:

```svelte
<script>
  let count = $state(0);
  let doubled = $derived(count * 2);
  
  // ❌ Error! Can't assign to derived
  // doubled = 10;
  
  // ✅ Change the source instead
  count = 5; // doubled automatically becomes 10
</script>
```

📖 [$derived documentation](https://svelte.dev/docs/svelte/$derived)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*