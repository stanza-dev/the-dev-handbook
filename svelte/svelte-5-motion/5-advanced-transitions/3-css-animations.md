---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-css-animations"
---

# Beyond Transitions: CSS Keyframes

For complex animations that don't involve entering/leaving, use CSS keyframes.

## Basic CSS Animation in Svelte

```svelte
<script>
  let isAnimating = $state(false);
</script>

<div class="box" class:pulse={isAnimating}>
  Box
</div>

<button onclick={() => isAnimating = !isAnimating}>
  Toggle Animation
</button>

<style>
  .box {
    width: 100px;
    height: 100px;
    background: #ff3e00;
  }
  
  .pulse {
    animation: pulse 1s ease-in-out infinite;
  }
  
  @keyframes pulse {
    0%, 100% { transform: scale(1); }
    50% { transform: scale(1.1); }
  }
</style>
```

## Trigger Animation on Event

```svelte
<script>
  let key = $state(0);
  
  function triggerAnimation() {
    key++; // Force re-render to replay animation
  }
</script>

{#key key}
  <div class="bounce">Bounce!</div>
{/key}

<button onclick={triggerAnimation}>Bounce Again</button>

<style>
  .bounce {
    animation: bounce 0.5s ease-out;
  }
  
  @keyframes bounce {
    0% { transform: translateY(-50px); opacity: 0; }
    60% { transform: translateY(10px); }
    100% { transform: translateY(0); opacity: 1; }
  }
</style>
```

## Animation with Runes

```svelte
<script>
  let rotation = $state(0);
  let isSpinning = $state(false);
  
  $effect(() => {
    if (!isSpinning) return;
    
    const frame = requestAnimationFrame(function animate() {
      rotation = (rotation + 5) % 360;
      if (isSpinning) {
        requestAnimationFrame(animate);
      }
    });
    
    return () => cancelAnimationFrame(frame);
  });
</script>

<div style="transform: rotate({rotation}deg)">
  🌀
</div>

<button onclick={() => isSpinning = !isSpinning}>
  {isSpinning ? 'Stop' : 'Spin'}
</button>
```

## When to Use What

| Need | Use |
|------|-----|
| Enter/leave DOM | `transition:` |
| Continuous animation | CSS `@keyframes` |
| Value interpolation | `tweened`/`spring` |
| List reordering | `animate:flip` |
| Complex sequencing | CSS + events or Motion libs |

📖 [CSS animations](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Animations)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*