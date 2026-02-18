---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-stopping-propagation"
---

# Stopping Events: When and How

Sometimes you don't want events to bubble up. Let's learn how to control propagation effectively.

## stopPropagation()

This method stops the event from bubbling to parent elements:

```svelte
<script>
  function handleCardClick() {
    console.log('Opening card details...');
  }
  
  function handleDeleteClick(event) {
    event.stopPropagation(); // Don't trigger card click!
    console.log('Deleting card...');
  }
</script>

<div class="card" onclick={handleCardClick}>
  <h3>Card Title</h3>
  <p>Some content here...</p>
  
  <!-- Without stopPropagation, clicking delete would also open the card -->
  <button onclick={handleDeleteClick}>Delete</button>
</div>
```

## A Real-World Example: Modal

```svelte
<script>
  let showModal = $state(false);
  
  function closeModal() {
    showModal = false;
  }
  
  function handleModalContentClick(event) {
    event.stopPropagation(); // Don't close when clicking inside
  }
</script>

{#if showModal}
  <!-- Clicking the backdrop closes the modal -->
  <div class="modal-backdrop" onclick={closeModal}>
    <!-- But clicking the content shouldn't close it -->
    <div class="modal-content" onclick={handleModalContentClick}>
      <h2>Modal Title</h2>
      <p>Modal content...</p>
      <button onclick={closeModal}>Close</button>
    </div>
  </div>
{/if}
```

## stopImmediatePropagation()

This is more aggressive - it stops the event from reaching ANY other handlers, even on the same element:

```svelte
<script>
  function handler1(event) {
    console.log('Handler 1');
    event.stopImmediatePropagation();
  }
  
  function handler2(event) {
    console.log('Handler 2'); // Never runs!
  }
</script>
```

## Best Practices

**Be cautious with stopPropagation:**

1. **It breaks other listeners** - Analytics tools, keyboard shortcuts, and other handlers might rely on bubbling
2. **It's invisible** - Hard to debug when events "mysteriously" don't work
3. **Alternative: Check target** - Sometimes you can check `event.target` instead:

```svelte
<script>
  function handleBackdropClick(event) {
    // Only close if clicking the backdrop itself, not its children
    if (event.target === event.currentTarget) {
      closeModal();
    }
  }
</script>

<div class="backdrop" onclick={handleBackdropClick}>
  <div class="modal">
    <!-- Clicks here won't close the modal -->
  </div>
</div>
```

📖 [Event handling documentation](https://svelte.dev/docs/svelte/basic-markup)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*