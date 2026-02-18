---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-multiple-handlers"
---

# Attaching Multiple Handlers

Sometimes you need to do several things when an event occurs. Svelte 5 provides clean patterns for this.

## Combining Logic in One Handler

The simplest approach is to call multiple functions from a single handler:

```svelte
<script>
  function logEvent(event) {
    console.log('Event occurred:', event.type);
  }
  
  function trackAnalytics() {
    analytics.track('button_clicked');
  }
  
  function performAction() {
    // The main action
  }
  
  function handleClick(event) {
    logEvent(event);
    trackAnalytics();
    performAction();
  }
</script>

<button onclick={handleClick}>Do something</button>
```

## Inline Composition

For simpler cases, compose directly in the template:

```svelte
<script>
  let count = $state(0);
  
  function log(msg) {
    console.log(msg);
  }
</script>

<button onclick={(e) => {
  log('Button clicked');
  count++;
}}>
  Clicked {count} times
</button>
```

## Creating Handler Composers

For reusable patterns, create a utility function:

```svelte
<script>
  // Utility to compose multiple handlers
  function compose(...handlers) {
    return (event) => {
      for (const handler of handlers) {
        handler(event);
      }
    };
  }
  
  function withLogging(handler) {
    return (event) => {
      console.log(`Event: ${event.type}`);
      handler(event);
    };
  }
  
  function handleClick() {
    console.log('Main action');
  }
  
  function trackClick() {
    analytics.track('click');
  }
</script>

<button onclick={compose(
  withLogging(handleClick),
  trackClick
)}>
  Click me
</button>
```

## Conditional Event Handling

You can also conditionally attach different handlers:

```svelte
<script>
  let isAdmin = $state(false);
  
  function handleUserClick() {
    console.log('User action');
  }
  
  function handleAdminClick() {
    console.log('Admin action with extra powers');
  }
</script>

<button onclick={isAdmin ? handleAdminClick : handleUserClick}>
  Do Action
</button>
```

📖 [Event handlers documentation](https://svelte.dev/docs/svelte/basic-markup#Event-handlers)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*