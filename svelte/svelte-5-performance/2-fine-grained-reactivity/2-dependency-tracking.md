---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-dependency-tracking"
---

# How Svelte Tracks Dependencies

Svelte automatically knows what depends on what. Here's how it works.

## The Magic of Automatic Tracking

```svelte
<script>
  let firstName = $state('John');
  let lastName = $state('Doe');
  
  // Svelte tracks: fullName depends on firstName AND lastName
  let fullName = $derived(`${firstName} ${lastName}`);
  
  // Svelte tracks: greeting depends on fullName
  let greeting = $derived(`Hello, ${fullName}!`);
</script>

<!-- Svelte tracks: this text depends on greeting -->
<p>{greeting}</p>
```

## Dependency Graph

```
firstName ────┐
              │
              ├──▶ fullName ──▶ greeting ──▶ <p> text node
              │
lastName  ────┘
```

## How Tracking Works

1. **During execution**, Svelte keeps a "current reactive context"
2. **When a signal is read**, it registers the current context as a subscriber
3. **When a signal changes**, it notifies only its subscribers

```javascript
// Conceptually:
let currentEffect = null;

$effect(() => {
  // Svelte sets: currentEffect = this effect
  console.log(count);  // count.get() sees currentEffect, adds it as subscriber
  // Svelte clears: currentEffect = null
});

// Later, when count changes:
count++;  // count.set() notifies all subscribers
```

## Conditional Dependencies

Dependencies are tracked per execution:

```svelte
<script>
  let showDetails = $state(false);
  let name = $state('Alice');
  let bio = $state('Developer');
  
  // Dependencies change based on condition!
  let display = $derived(
    showDetails ? `${name}: ${bio}` : name
  );
</script>
```

- When `showDetails = false`: display depends on `name` only
- When `showDetails = true`: display depends on `name` AND `bio`

Svelte re-tracks dependencies every time the derived runs!

📖 [Derived documentation](https://svelte.dev/docs/svelte/$derived)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*