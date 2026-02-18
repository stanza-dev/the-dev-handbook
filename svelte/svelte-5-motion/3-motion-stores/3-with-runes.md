---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-motion-with-runes"
---

# Motion Stores in Svelte 5

Motion stores work seamlessly with Svelte 5's runes system.

## Reading with $

Since `tweened` and `spring` are stores, use the `$` prefix:

```svelte
<script>
  import { tweened } from 'svelte/motion';
  
  const count = tweened(0);
  
  // $count gives you the current animated value
  let displayCount = $derived(Math.round($count));
</script>

<p>Count: {displayCount}</p>
```

## Reactive Updates with Effects

```svelte
<script>
  import { spring } from 'svelte/motion';
  
  let targetX = $state(0);
  let targetY = $state(0);
  
  const position = spring({ x: 0, y: 0 });
  
  // Update spring whenever targets change
  $effect(() => {
    position.set({ x: targetX, y: targetY });
  });
</script>

<input type="range" bind:value={targetX} min="0" max="500" />
<input type="range" bind:value={targetY} min="0" max="500" />

<div style="transform: translate({$position.x}px, {$position.y}px)" />
```

## Derived Motion Values

```svelte
<script>
  import { tweened } from 'svelte/motion';
  
  const baseValue = tweened(0);
  
  // Derive values from the animated store
  let doubled = $derived($baseValue * 2);
  let percentage = $derived(`${$baseValue}%`);
  let isComplete = $derived($baseValue >= 100);
</script>
```

## Animation on State Change

```svelte
<script>
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  
  let items = $state([1, 2, 3]);
  
  const listHeight = tweened(0, { duration: 300, easing: cubicOut });
  
  // Animate height when items change
  $effect(() => {
    listHeight.set(items.length * 50);
  });
</script>

<div style="height: {$listHeight}px; overflow: hidden;">
  {#each items as item}
    <div style="height: 50px">{item}</div>
  {/each}
</div>
```

📖 [Stores documentation](https://svelte.dev/docs/svelte/stores)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*