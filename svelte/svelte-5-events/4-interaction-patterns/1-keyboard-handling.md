---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-keyboard-handling"
---

# Mastering Keyboard Events

Keyboard interactions are essential for accessibility and power users. Let's learn how to handle them properly in Svelte 5.

## The Three Keyboard Events

```svelte
<script>
  function handleKeydown(event) {
    console.log('Key pressed down:', event.key);
  }
  
  function handleKeyup(event) {
    console.log('Key released:', event.key);
  }
  
  function handleKeypress(event) {
    // Deprecated! Avoid using
    console.log('Key pressed:', event.key);
  }
</script>

<input 
  onkeydown={handleKeydown}
  onkeyup={handleKeyup}
/>
```

**Use `onkeydown`** for most cases - it fires immediately when a key is pressed and repeats if held.

## Understanding KeyboardEvent Properties

```svelte
<script>
  function handleKey(event) {
    console.log('key:', event.key);       // 'Enter', 'a', 'ArrowUp'
    console.log('code:', event.code);     // 'Enter', 'KeyA', 'ArrowUp'
    console.log('keyCode:', event.keyCode); // 13 (deprecated, don't use)
    
    // Modifier keys
    console.log('shift:', event.shiftKey);  // true/false
    console.log('ctrl:', event.ctrlKey);    // true/false
    console.log('alt:', event.altKey);      // true/false
    console.log('meta:', event.metaKey);    // true/false (Cmd on Mac)
  }
</script>
```

**event.key** - The value produced by the key ("a", "A", "Enter")
**event.code** - The physical key on the keyboard ("KeyA" regardless of Shift)

## Common Keyboard Patterns

### Enter to Submit

```svelte
<script>
  let value = $state('');
  
  function handleKeydown(event) {
    if (event.key === 'Enter' && !event.shiftKey) {
      event.preventDefault();
      submit();
    }
  }
</script>

<textarea onkeydown={handleKeydown} bind:value></textarea>
```

### Escape to Close

```svelte
<script>
  let isOpen = $state(false);
  
  function handleKeydown(event) {
    if (event.key === 'Escape') {
      isOpen = false;
    }
  }
</script>

<svelte:window onkeydown={handleKeydown} />

{#if isOpen}
  <div class="modal">...</div>
{/if}
```

### Keyboard Shortcuts

```svelte
<script>
  function handleKeydown(event) {
    // Ctrl+S or Cmd+S to save
    if ((event.ctrlKey || event.metaKey) && event.key === 's') {
      event.preventDefault();
      save();
    }
    
    // Ctrl+K to open search
    if ((event.ctrlKey || event.metaKey) && event.key === 'k') {
      event.preventDefault();
      openSearch();
    }
  }
</script>

<svelte:window onkeydown={handleKeydown} />
```

### Arrow Key Navigation

```svelte
<script>
  let selectedIndex = $state(0);
  let items = ['Apple', 'Banana', 'Cherry'];
  
  function handleKeydown(event) {
    switch (event.key) {
      case 'ArrowDown':
        event.preventDefault();
        selectedIndex = Math.min(selectedIndex + 1, items.length - 1);
        break;
      case 'ArrowUp':
        event.preventDefault();
        selectedIndex = Math.max(selectedIndex - 1, 0);
        break;
      case 'Enter':
        selectItem(items[selectedIndex]);
        break;
    }
  }
</script>

<ul onkeydown={handleKeydown} tabindex="0">
  {#each items as item, i}
    <li class:selected={i === selectedIndex}>{item}</li>
  {/each}
</ul>
```

📖 [Special elements documentation](https://svelte.dev/docs/svelte/svelte-window)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*