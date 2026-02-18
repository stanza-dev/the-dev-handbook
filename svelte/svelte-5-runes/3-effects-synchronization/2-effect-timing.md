---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-effect-timing"
---

# Controlling When Effects Run

Sometimes the default timing of `$effect` (after DOM updates) isn't what you need.

## $effect.pre: Before DOM Updates

`$effect.pre` runs *before* Svelte updates the DOM. This is useful for reading DOM state that will change.

### Use Case: Maintaining Scroll Position

When adding items to a list, you might want to maintain the user's scroll position:

```js
let items = $state([]);

$effect.pre(() => {
  // Read scroll position BEFORE new items are rendered
  const list = document.querySelector('.list');
  if (list) {
    previousScrollTop = list.scrollTop;
    previousScrollHeight = list.scrollHeight;
  }
});

$effect(() => {
  // Adjust scroll AFTER items are rendered
  const list = document.querySelector('.list');
  if (list) {
    const newScrollHeight = list.scrollHeight;
    list.scrollTop = previousScrollTop + (newScrollHeight - previousScrollHeight);
  }
});
```

### Use Case: Autofocus

```js
let showInput = $state(false);
let inputElement;

$effect.pre(() => {
  // Element doesn't exist yet, just tracking state
  if (showInput) {
    // Will focus after DOM updates
  }
});

$effect(() => {
  if (showInput && inputElement) {
    inputElement.focus();  // DOM is ready now
  }
});
```

## $effect.root: Escaping Component Lifecycle

Normally, effects are tied to component lifecycle. `$effect.root` creates an effect that must be manually destroyed:

```js
const cleanup = $effect.root(() => {
  $effect(() => {
    console.log('This effect lives until cleanup() is called');
  });
});

// Later...
cleanup();  // Manually destroy the effect
```

### Use Case: Global Subscriptions

```js
// In a .svelte.js file
function createGlobalTimer() {
  let count = $state(0);
  
  const cleanup = $effect.root(() => {
    $effect(() => {
      const id = setInterval(() => count++, 1000);
      return () => clearInterval(id);
    });
  });
  
  return {
    get count() { return count; },
    destroy: cleanup
  };
}
```

## Effect Timing Summary

| Function | Runs | Use Case |
|----------|------|----------|
| `$effect` | After DOM updates | Most side effects |
| `$effect.pre` | Before DOM updates | Read DOM that will change |
| `$effect.root` | Manual control | Global state, testing |

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*