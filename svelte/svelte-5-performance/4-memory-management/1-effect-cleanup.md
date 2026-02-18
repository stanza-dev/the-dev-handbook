---
source_course: "svelte-5-performance"
source_lesson: "svelte-5-performance-effect-cleanup"
---

# Memory Leaks and Effect Cleanup

When you create subscriptions, timers, or event listeners in effects, you MUST clean them up to prevent memory leaks.

## The Problem: Leaked Listeners

```svelte
<script>
  let count = $state(0);
  
  // ❌ BAD: Listener never removed!
  $effect(() => {
    window.addEventListener('resize', () => {
      console.log('Resized!');
    });
  });
</script>
```

Every time this effect runs (or component remounts), a NEW listener is added. They stack up, causing:
- Memory leaks
- Multiple handlers firing
- Degraded performance

## The Solution: Return Cleanup Function

```svelte
<script>
  $effect(() => {
    const handler = () => console.log('Resized!');
    window.addEventListener('resize', handler);
    
    // ✅ GOOD: Return cleanup function
    return () => {
      window.removeEventListener('resize', handler);
    };
  });
</script>
```

## When Cleanup Runs

1. **When effect re-runs** (dependencies changed)
2. **When component unmounts**

```svelte
<script>
  let userId = $state(1);
  
  $effect(() => {
    console.log('Setting up for user', userId);
    const subscription = subscribeToUser(userId);
    
    return () => {
      console.log('Cleaning up for user', userId);
      subscription.unsubscribe();
    };
  });
  
  // If userId changes from 1 to 2:
  // 1. "Cleaning up for user 1"
  // 2. "Setting up for user 2"
</script>
```

## Common Cleanup Patterns

### Timers
```javascript
$effect(() => {
  const interval = setInterval(() => {
    console.log('tick');
  }, 1000);
  
  return () => clearInterval(interval);
});
```

### WebSocket
```javascript
$effect(() => {
  const ws = new WebSocket('wss://example.com');
  ws.onmessage = handleMessage;
  
  return () => ws.close();
});
```

### AbortController for fetch
```javascript
$effect(() => {
  const controller = new AbortController();
  
  fetch('/api/data', { signal: controller.signal })
    .then(r => r.json())
    .then(data => result = data);
  
  return () => controller.abort();
});
```

📖 [Effect documentation](https://svelte.dev/docs/svelte/$effect)

---

> 📘 *This lesson is part of the [Under the Hood & Performance](https://stanza.dev/courses/svelte-5-performance) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*