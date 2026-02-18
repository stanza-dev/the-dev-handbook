---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-effect-deep-dive"
---

# Understanding Side Effects

Effects let you synchronize your reactive state with external systems — the DOM, APIs, localStorage, browser APIs, etc.

## Basic $effect

```js
let count = $state(0);

$effect(() => {
  // This runs:
  // 1. After component mounts (initial run)
  // 2. Whenever 'count' changes
  // 3. After the DOM has updated
  
  document.title = `Count: ${count}`;
});
```

## The Effect Lifecycle

```js
$effect(() => {
  console.log('Effect running, count is:', count);
  
  // Cleanup function (optional)
  return () => {
    console.log('Cleanup running');
  };
});
```

Timeline:
1. Component mounts → Effect runs
2. `count` changes → Cleanup runs → Effect runs again
3. Component unmounts → Cleanup runs

## When Does $effect Run?

- **After DOM updates**: Safe to read/measure DOM
- **Asynchronously batched**: Multiple state changes = one effect run
- **In dependency order**: Effects respect the reactive graph

## Practical Examples

### LocalStorage Sync

```js
let theme = $state('light');

// Load from localStorage on mount
$effect(() => {
  const saved = localStorage.getItem('theme');
  if (saved) theme = saved;
});

// Save to localStorage on change
$effect(() => {
  localStorage.setItem('theme', theme);
});
```

### Event Listeners

```js
$effect(() => {
  function handleResize() {
    console.log('Window resized to', window.innerWidth);
  }
  
  window.addEventListener('resize', handleResize);
  
  return () => {
    window.removeEventListener('resize', handleResize);
  };
});
```

### API Calls

```js
let userId = $state(1);
let user = $state(null);
let loading = $state(false);

$effect(() => {
  loading = true;
  
  fetch(`/api/users/${userId}`)
    .then(r => r.json())
    .then(data => {
      user = data;
      loading = false;
    });
});
```

## Anti-Patterns to Avoid

### ❌ Don't Derive State in Effects

```js
// BAD: Use $derived instead!
$effect(() => {
  doubled = count * 2;  // This is wrong
});

// GOOD
let doubled = $derived(count * 2);
```

### ❌ Don't Create Infinite Loops

```js
// BAD: Infinite loop!
$effect(() => {
  count += 1;  // Reading AND writing count
});
```

### ❌ Don't Forget Cleanup

```js
// BAD: Memory leak!
$effect(() => {
  setInterval(() => console.log('tick'), 1000);
});

// GOOD
$effect(() => {
  const id = setInterval(() => console.log('tick'), 1000);
  return () => clearInterval(id);
});
```

## Resources

- [$effect Documentation](https://svelte.dev/docs/svelte/$effect) — Complete reference for side effects.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*