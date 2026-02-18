---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-spring-store"
---

# Springs: Natural, Physics-Based Motion

While `tweened` follows a fixed duration, `spring` uses physics simulation for more natural movement.

## Basic Spring Usage

```svelte
<script>
  import { spring } from 'svelte/motion';
  
  const coords = spring({ x: 0, y: 0 }, {
    stiffness: 0.1,  // How 'tight' the spring is (0-1)
    damping: 0.25    // How quickly oscillation stops (0-1)
  });
</script>

<svelte:window onmousemove={(e) => coords.set({ x: e.clientX, y: e.clientY })} />

<div 
  class="follower" 
  style="transform: translate({$coords.x}px, {$coords.y}px)"
/>
```

## Spring vs Tweened

| Feature | tweened | spring |
|---------|---------|--------|
| Control | Duration-based | Physics-based |
| Feel | Predictable | Natural, bouncy |
| Parameters | duration, easing | stiffness, damping |
| Best for | Progress bars, countdowns | Cursor followers, dragging |

## Spring Parameters

```javascript
const value = spring(initial, {
  stiffness: 0.1,  // Higher = faster snap (0.01 - 1)
  damping: 0.25,   // Higher = less bounce (0 - 1)
  precision: 0.01  // When to consider 'at rest'
});
```

### Parameter Effects:

- **High stiffness, high damping** = Quick, no bounce
- **Low stiffness, low damping** = Slow, very bouncy
- **High stiffness, low damping** = Quick, bouncy
- **Low stiffness, high damping** = Slow, smooth

## Practical Example: Draggable Element

```svelte
<script>
  import { spring } from 'svelte/motion';
  
  const position = spring({ x: 100, y: 100 });
  let isDragging = $state(false);
  let offset = { x: 0, y: 0 };
  
  function startDrag(event) {
    isDragging = true;
    offset = {
      x: event.clientX - $position.x,
      y: event.clientY - $position.y
    };
  }
  
  function drag(event) {
    if (!isDragging) return;
    position.set({
      x: event.clientX - offset.x,
      y: event.clientY - offset.y
    });
  }
  
  function endDrag() {
    isDragging = false;
  }
</script>

<svelte:window 
  onmousemove={drag} 
  onmouseup={endDrag} 
/>

<div
  class="draggable"
  style="transform: translate({$position.x}px, {$position.y}px)"
  onmousedown={startDrag}
>
  Drag me!
</div>
```

📖 [Spring documentation](https://svelte.dev/docs/svelte/motion#spring)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*