---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-custom-event-options"
---

# CustomEvent Configuration

Understanding the options you can pass to `CustomEvent` gives you fine control over event behavior.

## The CustomEvent Constructor

```javascript
new CustomEvent(eventName, options)
```

## Options Object

### bubbles (default: false)

Controls whether the event bubbles up through the DOM:

```javascript
// Won't bubble - only the target element receives it
new CustomEvent('local', { bubbles: false })

// Bubbles up - all ancestors can catch it
new CustomEvent('global', { bubbles: true })
```

### composed (default: false)

Controls whether the event crosses shadow DOM boundaries:

```javascript
// Stays within shadow DOM
new CustomEvent('internal', { composed: false })

// Crosses shadow DOM boundaries
new CustomEvent('external', { composed: true })
```

### cancelable (default: false)

Allows listeners to prevent the default behavior:

```javascript
const event = new CustomEvent('confirmdelete', {
  bubbles: true,
  cancelable: true,
  detail: { itemId: 123 }
});

element.dispatchEvent(event);

// Check if any listener called preventDefault()
if (!event.defaultPrevented) {
  actuallyDeleteItem(123);
}
```

```svelte
<!-- Listener can cancel -->
<div onconfirmdelete={(e) => {
  if (!confirm('Are you sure?')) {
    e.preventDefault(); // Cancel the deletion
  }
}}>
```

### detail

Your custom payload - can be any value:

```javascript
new CustomEvent('useraction', {
  detail: {
    action: 'purchase',
    item: { id: 1, name: 'Widget' },
    quantity: 5,
    metadata: { source: 'cart', timestamp: Date.now() }
  }
});
```

## Accessing Event Properties

```javascript
function handleCustomEvent(event) {
  // Standard event properties still work
  console.log(event.type);           // 'useraction'
  console.log(event.target);         // Element that dispatched
  console.log(event.currentTarget);  // Element with listener
  console.log(event.bubbles);        // true/false
  
  // Your custom data
  console.log(event.detail);         // { action: 'purchase', ... }
}
```

## TypeScript Typing

```typescript
// Define your event detail type
type ItemSelectedDetail = {
  id: number;
  name: string;
};

// Type the event handler
function handleItemSelected(event: CustomEvent<ItemSelectedDetail>) {
  const { id, name } = event.detail;
  console.log(`Selected: ${name} (${id})`);
}
```

📖 [MDN CustomEvent documentation](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*