---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-custom-event-patterns"
---

# Real-World CustomEvent Patterns

Let's explore practical patterns that make CustomEvents powerful in Svelte 5 applications.

## Pattern 1: Event Bus

Create a simple event bus for app-wide communication:

```svelte
<!-- eventBus.svelte.js -->
<script module>
  const target = new EventTarget();
  
  export function emit(eventName, detail) {
    target.dispatchEvent(new CustomEvent(eventName, { detail }));
  }
  
  export function on(eventName, handler) {
    target.addEventListener(eventName, handler);
    return () => target.removeEventListener(eventName, handler);
  }
</script>
```

```svelte
<!-- Component A - emits -->
<script>
  import { emit } from './eventBus.svelte.js';
</script>

<button onclick={() => emit('cart:updated', { count: 5 })}>
  Add to Cart
</button>
```

```svelte
<!-- Component B - listens -->
<script>
  import { on } from './eventBus.svelte.js';
  
  let cartCount = $state(0);
  
  $effect(() => {
    const unsubscribe = on('cart:updated', (e) => {
      cartCount = e.detail.count;
    });
    return unsubscribe; // Cleanup on unmount
  });
</script>

<span>Cart: {cartCount}</span>
```

## Pattern 2: Action-Based Events

Create a Svelte action that dispatches custom events:

```svelte
<script>
  // Action that detects long press
  function longpress(node, duration = 500) {
    let timer;
    
    function handleMouseDown() {
      timer = setTimeout(() => {
        node.dispatchEvent(new CustomEvent('longpress', {
          bubbles: true,
          detail: { duration }
        }));
      }, duration);
    }
    
    function handleMouseUp() {
      clearTimeout(timer);
    }
    
    node.addEventListener('mousedown', handleMouseDown);
    node.addEventListener('mouseup', handleMouseUp);
    node.addEventListener('mouseleave', handleMouseUp);
    
    return {
      destroy() {
        node.removeEventListener('mousedown', handleMouseDown);
        node.removeEventListener('mouseup', handleMouseUp);
        node.removeEventListener('mouseleave', handleMouseUp);
      }
    };
  }
</script>

<button use:longpress={1000} onlongpress={() => alert('Long pressed!')}>
  Hold me
</button>
```

## Pattern 3: Portal Communication

When components render in portals (like modals), CustomEvents bridge the gap:

```svelte
<!-- Modal content renders in a portal, outside normal tree -->
<script>
  function handleConfirm() {
    // Can't use callback props - we're in a portal!
    // CustomEvent bubbles through the document
    document.dispatchEvent(new CustomEvent('modal:confirm', {
      detail: { modalId: 'delete-user', confirmed: true }
    }));
  }
</script>
```

## Pattern 4: Combining with Effects

```svelte
<script>
  let element;
  
  // Listen for custom events on a specific element
  $effect(() => {
    if (!element) return;
    
    function handler(event) {
      console.log('Custom event received:', event.detail);
    }
    
    element.addEventListener('mycustomevent', handler);
    return () => element.removeEventListener('mycustomevent', handler);
  });
</script>

<div bind:this={element}>
  <ChildComponent />
</div>
```

📖 [Event handling documentation](https://svelte.dev/docs/svelte/basic-markup)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*