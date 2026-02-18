---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-inspect-rune"
---

# Debugging Reactivity with $inspect

Debugging reactive code can be tricky — when did this value change? Why? The `$inspect` rune is your answer.

## Basic $inspect

```js
let count = $state(0);
let doubled = $derived(count * 2);

$inspect(count);    // Logs whenever count changes
$inspect(doubled);  // Logs whenever doubled changes
```

Output in console:
```
init  0
update  1
update  2
```

## Inspect Multiple Values

```js
$inspect(count, doubled, user.name);
```

Logs all values together whenever ANY of them changes.

## Custom Handlers with .with()

For advanced debugging, provide a custom handler:

```js
$inspect(count).with((type, value) => {
  // type is 'init' or 'update'
  console.log(`Count ${type}: ${value}`);
});
```

## Triggering the Debugger

```js
$inspect(count).with((type, value) => {
  if (value > 10) {
    debugger;  // Opens DevTools debugger
  }
});
```

## Conditional Inspection

```js
$inspect(user).with((type, user) => {
  if (user?.role === 'admin') {
    console.log('Admin user detected:', user);
  }
});
```

## Development Only

**Important**: `$inspect` only works in development mode. In production builds, it's completely removed — no performance impact!

## Practical Examples

### Tracking State Changes

```js
let form = $state({
  email: '',
  password: '',
  remember: false
});

// See every form change
$inspect(form);
```

### Debugging Derived Calculations

```js
let items = $state([...]);
let total = $derived.by(() => {
  const result = items.reduce((sum, i) => sum + i.price, 0);
  return result;
});

// When does total recalculate?
$inspect(total);
```

### Debugging Effect Dependencies

```js
$effect(() => {
  // Add inspect at the start to see what triggers this effect
  $inspect('Effect triggered, count:', count, 'user:', user.name);
  
  // ... rest of effect
});
```

## Resources

- [$inspect Documentation](https://svelte.dev/docs/svelte/$inspect) — Official guide to debugging with $inspect.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*