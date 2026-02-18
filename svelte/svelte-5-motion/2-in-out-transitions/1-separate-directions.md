---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-separate-directions"
---

# Fine-Grained Control

Sometimes you want different animations for entering and leaving. Use `in:` and `out:` instead of `transition:`.

## Basic Usage

```svelte
<script>
  import { fly, fade, scale } from 'svelte/transition';
  
  let visible = $state(true);
</script>

{#if visible}
  <div
    in:fly={{ y: 200, duration: 500 }}
    out:fade={{ duration: 200 }}
  >
    Flies in dramatically, fades out quickly
  </div>
{/if}
```

## Why Separate?

| Scenario | In | Out |
|----------|----|----||
| Toast notification | Slide from top | Fade |
| Modal | Scale up | Fade fast |
| Sidebar | Slide | Slide (same) |
| Deleted item | - | Shrink + fade |

## transition: vs in:/out:

`transition:fn` is shorthand for `in:fn out:fn`:

```svelte
<!-- These are equivalent -->
<div transition:fade>
<div in:fade out:fade>
```

## Different Configurations

You can use the same transition with different parameters:

```svelte
<div
  in:fly={{ y: 100, duration: 600, easing: elasticOut }}
  out:fly={{ y: -50, duration: 200, easing: cubicIn }}
>
  Bouncy entrance, quick exit upward
</div>
```

## Intro Only or Outro Only

You can have animation in only one direction:

```svelte
<!-- Only animates when appearing -->
<div in:fade>
  Fades in, instant disappear
</div>

<!-- Only animates when leaving -->
<div out:fly={{ y: 100 }}>
  Instant appear, flies out
</div>
```

📖 [Transitions documentation](https://svelte.dev/docs/svelte/transition)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*