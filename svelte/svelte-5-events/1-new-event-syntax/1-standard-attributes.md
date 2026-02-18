---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-standard-attributes"
---

# Goodbye `on:` - Hello Standard HTML

One of the most visible changes in Svelte 5 is how we handle events. The framework has moved away from Svelte-specific syntax toward standard HTML attributes, making your code more familiar to anyone who knows JavaScript.

## What Changed?

In Svelte 4, we used a special `on:` directive to attach event handlers:

```svelte
<!-- Svelte 4 (OLD WAY - don't use!) -->
<button on:click={handleClick}>Click me</button>
<input on:input={handleInput} />
<form on:submit={handleSubmit}>...</form>
```

In Svelte 5, we use standard HTML event attributes - the same ones you'd use in vanilla JavaScript:

```svelte
<!-- Svelte 5 (NEW WAY) -->
<button onclick={handleClick}>Click me</button>
<input oninput={handleInput} />
<form onsubmit={handleSubmit}>...</form>
```

## Why This Change?

This shift brings several benefits:

**1. Familiar to All JavaScript Developers**
Anyone who knows HTML and JavaScript already knows how to use `onclick`, `onmouseover`, `onkeydown`, etc. There's nothing Svelte-specific to learn.

**2. Perfect TypeScript Support**
Because these are standard DOM attributes, TypeScript understands them natively. Your IDE will autocomplete event names and show the correct event types:

```svelte
<script lang="ts">
  // TypeScript knows this is a MouseEvent
  function handleClick(event: MouseEvent) {
    console.log(event.clientX, event.clientY);
  }
  
  // And this is an InputEvent
  function handleInput(event: InputEvent & { currentTarget: HTMLInputElement }) {
    console.log(event.currentTarget.value);
  }
</script>

<button onclick={handleClick}>Click</button>
<input oninput={handleInput} />
```

**3. Works with All DOM Events**
Every event that exists in the DOM is available. Common ones include:

- **Mouse:** `onclick`, `ondblclick`, `onmouseenter`, `onmouseleave`, `onmousemove`
- **Keyboard:** `onkeydown`, `onkeyup`, `onkeypress`
- **Form:** `onsubmit`, `oninput`, `onchange`, `onfocus`, `onblur`
- **Touch:** `ontouchstart`, `ontouchmove`, `ontouchend`
- **Drag:** `ondragstart`, `ondragover`, `ondrop`

📖 [Event handlers documentation](https://svelte.dev/docs/svelte/basic-markup#Event-handlers)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*