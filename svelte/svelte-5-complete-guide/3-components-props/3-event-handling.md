---
source_course: "svelte-5-complete-guide"
source_lesson: "svelte-5-complete-guide-event-handling"
---

# The New Event Syntax

Svelte 5 brings a major simplification to event handling: **standard HTML event attributes**.

## Basic Events

Instead of the old `on:click` directive, use `onclick` (just like vanilla HTML):

```svelte
<script>
  let count = $state(0);

  function handleClick() {
    count++;
  }
</script>

<!-- Svelte 5 way -->
<button onclick={handleClick}>Clicked {count} times</button>

<!-- Or with inline arrow function -->
<button onclick={() => count++}>Clicked {count} times</button>
```

## Common Events

| Event | Usage |
|-------|-------|
| Click | `onclick={handler}` |
| Input | `oninput={handler}` |
| Change | `onchange={handler}` |
| Submit | `onsubmit={handler}` |
| Keydown | `onkeydown={handler}` |
| Mouse Enter | `onmouseenter={handler}` |
| Focus | `onfocus={handler}` |

## Event Object

The handler receives the standard DOM event object:

```svelte
<script>
  function handleInput(event) {
    console.log('You typed:', event.target.value);
  }
  
  function handleKeydown(event) {
    if (event.key === 'Enter') {
      console.log('Enter pressed!');
    }
  }
</script>

<input oninput={handleInput} onkeydown={handleKeydown} />
```

## Preventing Defaults

In Svelte 4, you could use `|preventDefault`. In Svelte 5, do it in your function:

```svelte
<script>
  function handleSubmit(event) {
    event.preventDefault();  // Prevent form submission
    console.log('Form submitted!');
  }
</script>

<form onsubmit={handleSubmit}>
  <input name="email" />
  <button>Submit</button>
</form>
```

## Component Events (Callback Props)

In Svelte 5, components emit events by calling **callback props** — functions passed from the parent:

```svelte
<!-- DeleteButton.svelte -->
<script>
  let { ondelete } = $props();
</script>

<button onclick={() => ondelete()}>Delete</button>
```

```svelte
<!-- App.svelte -->
<script>
  import DeleteButton from './DeleteButton.svelte';
  
  function handleDelete() {
    console.log('Item deleted!');
  }
</script>

<DeleteButton ondelete={handleDelete} />
```

This replaces the old `createEventDispatcher` pattern and is much simpler!

## Passing Data Back to Parent

```svelte
<!-- RatingStars.svelte -->
<script>
  let { onrate } = $props();
</script>

{#each [1, 2, 3, 4, 5] as star}
  <button onclick={() => onrate(star)}>⭐</button>
{/each}
```

```svelte
<!-- App.svelte -->
<script>
  import RatingStars from './RatingStars.svelte';
  
  function handleRating(stars) {
    console.log(`User rated: ${stars} stars`);
  }
</script>

<RatingStars onrate={handleRating} />
```

## Resources

- [Event Handling](https://svelte.dev/docs/svelte/v5-migration-guide#Event-changes) — Guide to the new event syntax in Svelte 5.

---

> 📘 *This lesson is part of the [Svelte 5: The Complete Guide](https://stanza.dev/courses/svelte-5-complete-guide) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*