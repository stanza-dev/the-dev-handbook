---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-event-delegation"
---

# Event Delegation: One Handler for Many Elements

Instead of attaching an event handler to every item in a list, you can attach ONE handler to the parent. This is called **event delegation**, and it's a powerful performance optimization.

## The Problem with Individual Handlers

Imagine you have a list of 1000 items, each with a click handler:

```svelte
<!-- INEFFICIENT: 1000 event listeners! -->
{#each items as item}
  <li onclick={() => handleClick(item.id)}>{item.name}</li>
{/each}
```

This creates 1000 separate event listeners. That's a lot of memory and can slow down your app.

## The Solution: Delegate to a Parent

Instead, put ONE handler on the parent and figure out which item was clicked:

```svelte
<script>
  let items = $state([
    { id: 1, name: 'Apple' },
    { id: 2, name: 'Banana' },
    { id: 3, name: 'Cherry' }
  ]);
  
  function handleListClick(event) {
    // Find the closest <li> element from wherever we clicked
    const li = event.target.closest('li');
    if (!li) return; // Clicked somewhere else in the list
    
    const id = li.dataset.id;
    console.log('Clicked item:', id);
  }
</script>

<ul onclick={handleListClick}>
  {#each items as item}
    <li data-id={item.id}>{item.name}</li>
  {/each}
</ul>
```

## Why This Works

1. When you click an `<li>`, the event bubbles up to the `<ul>`
2. Our handler on `<ul>` catches the event
3. We use `event.target.closest('li')` to find which `<li>` was clicked
4. We read the data from `data-id` attribute

## Using `closest()` Safely

The `closest()` method is essential for delegation. It traverses up the DOM tree:

```svelte
<script>
  function handleClick(event) {
    // closest() finds the nearest ancestor (or self) matching the selector
    const card = event.target.closest('.card');
    
    if (card) {
      // We clicked somewhere inside a card
      console.log('Card ID:', card.dataset.cardId);
    }
  }
</script>

<div onclick={handleClick} class="card-container">
  {#each cards as card}
    <div class="card" data-card-id={card.id}>
      <h3>{card.title}</h3>
      <p>{card.description}</p>
      <button>Action</button>
    </div>
  {/each}
</div>
```

Even if the user clicks the `<button>`, `<h3>`, or `<p>`, `closest('.card')` will find the parent card element.

## When to Use Delegation

✅ **Good for:**
- Lists with many items
- Dynamically added elements
- Reducing memory usage
- Handling events on elements that don't exist yet

❌ **Overkill for:**
- A few static buttons
- Cases where you need different handlers per element
- Forms with complex validation

📖 [Event handling documentation](https://svelte.dev/docs/svelte/basic-markup)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*