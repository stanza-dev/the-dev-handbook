---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-dispatch-custom"
---

# When to Use CustomEvent

While callback props are the preferred way to communicate between components in Svelte 5, sometimes you need events to bubble through multiple DOM layers without passing callbacks through each one. That's where native `CustomEvent` shines.

## The Problem: Deep Component Trees

```
<GrandParent>
  <Parent>
    <Child onaction={...}>
      <GrandChild onaction={...}>
        <Button /> ← Event originates here
```

With callback props, you'd need to thread `onaction` through every level. With CustomEvent, the event bubbles up naturally.

## Creating a CustomEvent

```svelte
<script>
  function handleClick(event) {
    // Create a custom event
    const customEvent = new CustomEvent('itemselected', {
      bubbles: true,      // Allow it to bubble up
      detail: {           // Your custom data
        id: 123,
        name: 'Selected Item'
      }
    });
    
    // Dispatch it from the current element
    event.currentTarget.dispatchEvent(customEvent);
  }
</script>

<button onclick={handleClick}>Select</button>
```

## Listening for Custom Events

Parent components can listen just like any other event:

```svelte
<script>
  function handleItemSelected(event) {
    console.log('Item selected:', event.detail);
    // { id: 123, name: 'Selected Item' }
  }
</script>

<!-- Svelte 5 treats 'on' + event name as an event listener -->
<div onitemselected={handleItemSelected}>
  <DeepNestedComponent />
</div>
```

## Complete Example: Notification System

```svelte
<!-- Button.svelte (deep in the tree) -->
<script>
  function notify(message, type = 'info') {
    const event = new CustomEvent('notify', {
      bubbles: true,
      detail: { message, type, timestamp: Date.now() }
    });
    document.dispatchEvent(event);
  }
</script>

<button onclick={() => notify('Item saved!', 'success')}>
  Save
</button>
```

```svelte
<!-- App.svelte (at the top) -->
<script>
  let notifications = $state([]);
  
  function handleNotify(event) {
    notifications = [...notifications, event.detail];
    
    // Auto-remove after 3 seconds
    setTimeout(() => {
      notifications = notifications.filter(n => n !== event.detail);
    }, 3000);
  }
</script>

<svelte:document onnotify={handleNotify} />

<div class="notifications">
  {#each notifications as { message, type }}
    <div class="notification {type}">{message}</div>
  {/each}
</div>
```

## When to Use CustomEvent vs Callback Props

| Use Case | Recommendation |
|----------|----------------|
| Parent-child communication | Callback props |
| Deeply nested events | CustomEvent |
| Global notifications | CustomEvent on document |
| Library/component that doesn't know its context | CustomEvent |
| Type-safe APIs | Callback props |

📖 [Event handling documentation](https://svelte.dev/docs/svelte/basic-markup)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*