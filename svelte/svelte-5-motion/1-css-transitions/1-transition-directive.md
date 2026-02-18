---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-transition-directive"
---

# Built-in Transition Magic

Svelte provides an elegant way to animate elements entering and leaving the DOM. The `transition:` directive makes complex animations trivially simple.

## Basic Usage

```svelte
<script>
  import { fade, fly, slide, scale, blur } from 'svelte/transition';
  
  let visible = $state(true);
</script>

<button onclick={() => visible = !visible}>Toggle</button>

{#if visible}
  <div transition:fade>
    I fade in and out
  </div>
{/if}
```

## Built-in Transitions

Svelte includes several ready-to-use transitions:

| Transition | Effect |
|------------|--------|
| `fade` | Opacity from 0 to 1 |
| `fly` | Moves from offset position + fades |
| `slide` | Slides in/out (like accordion) |
| `scale` | Scales from 0 to 1 + fades |
| `blur` | Blurs and fades |
| `draw` | SVG path drawing animation |

## Configuring Transitions

Pass parameters to customize behavior:

```svelte
{#if visible}
  <div transition:fly={{ 
    y: 200,        // Fly from 200px below
    duration: 800, // Take 800ms
    delay: 100,    // Wait 100ms before starting
    easing: quintOut // Use quintic easing
  }}>
    Flying content
  </div>
{/if}
```

## Common Parameters

Most transitions accept:
- `duration` - Animation length in milliseconds
- `delay` - Time to wait before starting
- `easing` - Easing function (from `svelte/easing`)

## How It Works

Transitions run via CSS animations, so they:
- Don't block the main thread
- Are GPU-accelerated
- Continue smoothly even during heavy JS work

```
Element enters DOM
       ↓
transition:fade runs
       ↓
Svelte generates CSS: opacity 0 → 1
       ↓
Browser animates via CSS (performant!)
```

📖 [Transitions documentation](https://svelte.dev/docs/svelte/transition)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*