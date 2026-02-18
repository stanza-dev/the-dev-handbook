---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-tweened-store"
---

# Smoothly Animated Values

While transitions handle elements entering/leaving, `tweened` handles smooth value changes over time.

## Basic Usage

```svelte
<script>
  import { tweened } from 'svelte/motion';
  
  const progress = tweened(0, {
    duration: 400,
    easing: cubicOut
  });
</script>

<progress value={$progress} max="100"></progress>

<button onclick={() => progress.set(100)}>Complete</button>
<button onclick={() => progress.set(0)}>Reset</button>
```

When you call `progress.set(100)`, the value smoothly animates from 0 to 100.

## Tweened is a Store

In Svelte 5, `tweened` returns a store. Use `$` prefix to read the current animated value:

```svelte
<script>
  const position = tweened({ x: 0, y: 0 });
</script>

<!-- Read current animated values -->
<div style="transform: translate({$position.x}px, {$position.y}px)">
  Moving box
</div>

<button onclick={() => position.set({ x: 200, y: 100 })}>
  Move
</button>
```

## Configuration Options

```javascript
const value = tweened(initialValue, {
  duration: 400,    // Animation duration in ms
  easing: cubicOut, // Easing function
  interpolate: (a, b) => (t) => ... // Custom interpolation
});
```

## Overriding Options Per-Update

```javascript
// Default: 400ms
const progress = tweened(0, { duration: 400 });

// Fast update
progress.set(50, { duration: 100 });

// Slow update
progress.set(100, { duration: 1000 });
```

## Animating Colors

```svelte
<script>
  import { tweened } from 'svelte/motion';
  import { interpolateLab } from 'd3-interpolate';
  
  const color = tweened('#ff3e00', {
    duration: 800,
    interpolate: interpolateLab // Smooth color interpolation
  });
</script>

<div style="background: {$color}" />

<button onclick={() => color.set('#40b3ff')}>Blue</button>
<button onclick={() => color.set('#ff3e00')}>Orange</button>
```

📖 [Motion documentation](https://svelte.dev/docs/svelte/motion)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*