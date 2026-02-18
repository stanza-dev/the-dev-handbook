---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-missing-modifiers"
---

# Where Are the Modifiers?

If you're coming from Svelte 4, you might miss event modifiers like `|preventDefault`, `|stopPropagation`, `|once`, and `|capture`. In Svelte 5, these are gone - but that's actually a good thing!

## Svelte 4 Modifiers (No Longer Supported)

```svelte
<!-- OLD Svelte 4 syntax - DON'T USE -->
<form on:submit|preventDefault={handleSubmit}>
<div on:click|stopPropagation={handleClick}>
<button on:click|once={handleFirstClick}>
<div on:scroll|capture={handleScroll}>
```

## The Svelte 5 Way: Just Use JavaScript

Modifiers were essentially shortcuts for things you can easily do in JavaScript. Now you write them explicitly:

### preventDefault()

Prevents the browser's default action (like form submission or link navigation):

```svelte
<script>
  function handleSubmit(event) {
    event.preventDefault(); // Stop the form from refreshing the page
    // Your form logic here
    console.log('Form submitted without page refresh');
  }
</script>

<form onsubmit={handleSubmit}>
  <input name="email" type="email" />
  <button type="submit">Subscribe</button>
</form>
```

### stopPropagation()

Stops the event from bubbling up to parent elements:

```svelte
<script>
  function handleCardClick() {
    console.log('Card clicked - opening details');
  }
  
  function handleButtonClick(event) {
    event.stopPropagation(); // Don't trigger card click
    console.log('Button clicked - deleting item');
  }
</script>

<div onclick={handleCardClick} class="card">
  <h3>Item Title</h3>
  <button onclick={handleButtonClick}>Delete</button>
</div>
```

### Implementing 'once' Behavior

For handlers that should only run once, track it with state:

```svelte
<script>
  let hasBeenClicked = $state(false);
  
  function handleOnce() {
    if (hasBeenClicked) return;
    hasBeenClicked = true;
    console.log('This only runs once!');
  }
</script>

<button onclick={handleOnce} disabled={hasBeenClicked}>
  Click me (once only)
</button>
```

Or use the native `addEventListener` with `{ once: true }` in an effect:

```svelte
<script>
  let button;
  
  $effect(() => {
    button?.addEventListener('click', () => {
      console.log('Only fires once!');
    }, { once: true });
  });
</script>

<button bind:this={button}>One-time click</button>
```

## Benefits of This Approach

1. **No magic syntax to learn** - just standard JavaScript
2. **More flexibility** - combine multiple behaviors easily
3. **Easier debugging** - you can log, add breakpoints, or conditionally apply
4. **Same code everywhere** - works identically in Svelte, React, Vue, or vanilla JS

📖 [Element directives documentation](https://svelte.dev/docs/svelte/basic-markup)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*