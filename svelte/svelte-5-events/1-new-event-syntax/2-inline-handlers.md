---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-inline-handlers"
---

# Writing Inline Event Handlers

You don't always need to define a separate function for your event handlers. Svelte 5 lets you write inline arrow functions directly in the attribute - perfect for simple operations.

## Simple Inline Handlers

For quick, one-line operations, inline handlers keep your code concise:

```svelte
<script>
  let count = $state(0);
  let name = $state('');
</script>

<!-- Simple increment -->
<button onclick={() => count++}>+1</button>
<button onclick={() => count--}>-1</button>

<!-- Direct state update -->
<input oninput={(e) => name = e.currentTarget.value} />

<!-- Toggle boolean -->
<button onclick={() => isOpen = !isOpen}>
  {isOpen ? 'Close' : 'Open'}
</button>
```

## Accessing Event Properties

The event object is automatically passed to your handler. You can destructure it inline:

```svelte
<script>
  let position = $state({ x: 0, y: 0 });
</script>

<!-- Access mouse coordinates -->
<div onmousemove={(e) => position = { x: e.clientX, y: e.clientY }}>
  Mouse: {position.x}, {position.y}
</div>

<!-- Get key pressed -->
<input onkeydown={(e) => {
  if (e.key === 'Enter') submitForm();
}} />
```

## When to Use Named Functions

While inline handlers are convenient, use named functions when:

1. **The logic is complex** (more than one line)
2. **You need to reuse the handler** across multiple elements
3. **You want better readability** for complex operations
4. **Testing is important** - named functions are easier to test

```svelte
<script>
  let items = $state([]);
  
  // Named function for complex logic
  function handleKeydown(event) {
    if (event.key === 'Enter' && !event.shiftKey) {
      event.preventDefault();
      addItem();
    } else if (event.key === 'Escape') {
      clearInput();
    }
  }
  
  // Simple actions can stay inline below
</script>

<input onkeydown={handleKeydown} />
<button onclick={() => items = []}>Clear All</button>
```

## Inline Handlers with Parameters

When you need to pass additional data to a handler, wrap it in an arrow function:

```svelte
<script>
  let items = $state(['Apple', 'Banana', 'Cherry']);
  
  function removeItem(index) {
    items = items.filter((_, i) => i !== index);
  }
</script>

{#each items as item, i}
  <div>
    {item}
    <button onclick={() => removeItem(i)}>×</button>
  </div>
{/each}
```

📖 [Event handlers documentation](https://svelte.dev/docs/svelte/basic-markup#Event-handlers)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*