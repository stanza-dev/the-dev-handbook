---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-easing-functions"
---

# Easing: The Soul of Animation

Easing functions control how animations accelerate and decelerate. The right easing makes animations feel natural.

## Available Easing Functions

```svelte
<script>
  import { 
    linear,
    cubicIn, cubicOut, cubicInOut,
    quadIn, quadOut, quadInOut,
    quartIn, quartOut, quartInOut,
    quintIn, quintOut, quintInOut,
    sineIn, sineOut, sineInOut,
    expoIn, expoOut, expoInOut,
    circIn, circOut, circInOut,
    backIn, backOut, backInOut,
    elasticIn, elasticOut, elasticInOut,
    bounceIn, bounceOut, bounceInOut
  } from 'svelte/easing';
</script>
```

## Understanding In/Out/InOut

- **In** - Starts slow, ends fast (acceleration)
- **Out** - Starts fast, ends slow (deceleration)
- **InOut** - Slow at both ends (natural motion)

```
linear:    ────────────────────────
cubicIn:   ━━━━━━━━━━━━────────────
cubicOut:  ────────────━━━━━━━━━━━━
cubicInOut: ━━━━────────────━━━━━━
```

## Using Easing

```svelte
<script>
  import { fly } from 'svelte/transition';
  import { elasticOut, backOut } from 'svelte/easing';
</script>

{#if visible}
  <!-- Bouncy entrance -->
  <div transition:fly={{ y: 50, easing: elasticOut }}>
    Bouncy!
  </div>
  
  <!-- Overshoots slightly before settling -->
  <div transition:fly={{ y: 50, easing: backOut }}>
    Back easing!
  </div>
{/if}
```

## Common Recommendations

| Use Case | Recommended Easing |
|----------|-------------------|
| UI elements appearing | `cubicOut`, `quintOut` |
| Modal popups | `backOut` |
| Playful animations | `elasticOut`, `bounceOut` |
| Natural movement | `cubicInOut` |
| Urgent notifications | `expoOut` |

## Custom Easing Function

```javascript
// Easing is just a function: (t: 0-1) => 0-1
function myEasing(t) {
  // Custom curve
  return t * t * (3 - 2 * t);
}
```

📖 [Easing documentation](https://svelte.dev/docs/svelte/easing)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*