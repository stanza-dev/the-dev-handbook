---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-event-phases"
---

# Understanding How Events Travel

Before diving into advanced patterns, you need to understand how events actually move through the DOM. This knowledge is fundamental to working with any web framework.

## The Three Phases of Event Flow

When you click a button nested inside a div inside the body, the event doesn't just happen on the button. It travels through the entire DOM:

```
          ┌─────────────────────────────────────────┐
          │           1. CAPTURE PHASE              │
          │         (top-down: window → target)     │
          └─────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────┐
│  window → document → html → body → div → button     │
└─────────────────────────────────────────────────────┘
                              │
                              ▼
          ┌─────────────────────────────────────────┐
          │           2. TARGET PHASE               │
          │         (event reaches target)          │
          └─────────────────────────────────────────┘
                              │
                              ▼
          ┌─────────────────────────────────────────┐
          │           3. BUBBLE PHASE               │
          │         (bottom-up: target → window)    │
          └─────────────────────────────────────────┘
```

**Capture Phase:** The event travels down from the window to the target element.

**Target Phase:** The event arrives at the element that was actually clicked.

**Bubble Phase:** The event travels back up to the window.

## Which Phase Do We Usually Use?

By default, event handlers listen to the **bubble phase**. This is what happens when you use `onclick`:

```svelte
<script>
  function handleOuter() {
    console.log('Outer clicked');
  }
  
  function handleInner() {
    console.log('Inner clicked');
  }
</script>

<div onclick={handleOuter}>
  <button onclick={handleInner}>Click me</button>
</div>

<!-- When you click the button, you'll see:
     "Inner clicked"
     "Outer clicked"
     (because the event bubbles UP)
-->
```

## Event Object Properties

The event object tells you exactly what's happening:

```svelte
<script>
  function handleClick(event) {
    console.log('target:', event.target);          // Element that was clicked
    console.log('currentTarget:', event.currentTarget); // Element with the handler
    console.log('eventPhase:', event.eventPhase);  // 1=capture, 2=target, 3=bubble
  }
</script>
```

- **`event.target`** - The actual element clicked (could be a child)
- **`event.currentTarget`** - The element your handler is attached to

This distinction is crucial for event delegation (coming next)!

📖 [Event handling documentation](https://svelte.dev/docs/svelte/basic-markup#Event-handlers)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*