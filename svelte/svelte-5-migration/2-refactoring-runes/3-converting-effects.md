---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-converting-effects"
---

# Side Effects with $effect

Svelte 4's `$:` was used for both derived values AND side effects. Svelte 5 separates these concerns.

## Pure Side Effects

```svelte
<!-- Svelte 4 -->
<script>
  let count = 0;
  
  $: {
    console.log('Count changed to', count);
  }
</script>

<!-- Svelte 5 -->
<script>
  let count = $state(0);
  
  $effect(() => {
    console.log('Count changed to', count);
  });
</script>
```

## DOM Manipulation

```svelte
<!-- Svelte 4 -->
<script>
  let canvas;
  let color = 'red';
  
  $: if (canvas) {
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = color;
    ctx.fillRect(0, 0, 100, 100);
  }
</script>

<!-- Svelte 5 -->
<script>
  let canvas;
  let color = $state('red');
  
  $effect(() => {
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    ctx.fillStyle = color;
    ctx.fillRect(0, 0, 100, 100);
  });
</script>
```

## Cleanup

```svelte
<!-- Svelte 4 -->
<script>
  import { onDestroy } from 'svelte';
  
  let interval;
  
  $: {
    clearInterval(interval);
    interval = setInterval(() => count++, 1000);
  }
  
  onDestroy(() => clearInterval(interval));
</script>

<!-- Svelte 5 (cleaner!) -->
<script>
  let count = $state(0);
  
  $effect(() => {
    const interval = setInterval(() => count++, 1000);
    return () => clearInterval(interval); // Automatic cleanup!
  });
</script>
```

## onMount Replacement

```svelte
<!-- Svelte 4 -->
<script>
  import { onMount } from 'svelte';
  
  onMount(() => {
    console.log('Mounted!');
    return () => console.log('Unmounting!');
  });
</script>

<!-- Svelte 5 -->
<script>
  $effect(() => {
    console.log('Mounted!');
    return () => console.log('Unmounting!');
  });
</script>
```

📖 [$effect documentation](https://svelte.dev/docs/svelte/$effect)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*