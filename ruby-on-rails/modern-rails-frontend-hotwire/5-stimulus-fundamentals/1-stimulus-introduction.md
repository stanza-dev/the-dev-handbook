---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-introduction"
---

# Controller Lifecycle

## Introduction
Stimulus controllers have a defined lifecycle tied to the DOM. When an element with `data-controller` appears in the page, Stimulus creates a controller instance and calls lifecycle callbacks. When the element is removed, the controller is torn down.

## Key Concepts
- **`connect()`**: Called when the controller element is inserted into the DOM.
- **`disconnect()`**: Called when the controller element is removed from the DOM.
- **`initialize()`**: Called once when the controller class is first instantiated (before connect).
- **`this.element`**: Reference to the DOM element the controller is attached to.

## Real World Context
Understanding the lifecycle is critical for managing resources. You start timers, open WebSocket connections, or initialize third-party libraries in `connect()`, and clean them up in `disconnect()`. Without proper cleanup, you get memory leaks and ghost event listeners.

## Deep Dive
### The Lifecycle Sequence

```
1. Element with data-controller appears in DOM
2. Stimulus finds the matching controller class
3. initialize() — called once (first time only)
4. connect() — called every time the element enters the DOM
5. ... controller is active ...
6. disconnect() — called when the element leaves the DOM
```

With Turbo Drive, elements frequently enter and leave the DOM as pages change. A controller on the current page is connected, then disconnected when Turbo navigates away, then reconnected if the user returns via the back button.

### Practical Example: Auto-Save

```javascript
// app/javascript/controllers/autosave_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    this.timer = setInterval(() => {
      this.element.requestSubmit()
    }, 30000) // Auto-save every 30 seconds
  }

  disconnect() {
    clearInterval(this.timer)
  }
}
```

The `connect()` method starts a timer. The `disconnect()` method clears it. Without the cleanup in `disconnect()`, navigating away would leave the timer running, causing errors when it tries to submit a form that no longer exists.

### Third-Party Library Integration

```javascript
import { Controller } from "@hotwired/stimulus"
import flatpickr from "flatpickr"

export default class extends Controller {
  connect() {
    this.picker = flatpickr(this.element, {
      dateFormat: "Y-m-d",
      enableTime: false
    })
  }

  disconnect() {
    this.picker.destroy()
  }
}
```

Initialize the library in `connect()` and destroy it in `disconnect()`. This pattern works for any JavaScript library: charts, maps, editors, date pickers.

## Common Pitfalls
1. **Forgetting disconnect cleanup** — Resources started in `connect()` must be cleaned up in `disconnect()`. Timers, event listeners, and library instances all need explicit teardown.
2. **Using initialize for DOM-dependent setup** — `initialize()` runs before the element may be fully in the DOM. Use `connect()` for anything that accesses DOM elements.

## Best Practices
1. **Always pair connect/disconnect** — If you create something in `connect()`, destroy it in `disconnect()`.
2. **Use connect for third-party libs** — Initialize external libraries in `connect()` so they work correctly with Turbo navigation.

## Summary
- `initialize()` runs once, `connect()` runs each time the element enters the DOM, `disconnect()` runs when it leaves.
- With Turbo, controllers connect and disconnect frequently as pages change.
- Always clean up resources (timers, listeners, libraries) in `disconnect()`.
- Use `connect()` (not `initialize()`) for DOM-dependent setup.

## Code Examples

**A controller demonstrating all lifecycle callbacks with proper resource cleanup in disconnect()**

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  initialize() {
    // Runs once when controller class is first used
    console.log("Controller class initialized")
  }

  connect() {
    // Runs each time the element enters the DOM
    console.log(`Connected to ${this.element.id}`)
    this.startPolling()
  }

  disconnect() {
    // Runs when element leaves DOM — clean up!
    this.stopPolling()
  }

  startPolling() {
    this.pollTimer = setInterval(() => this.refresh(), 5000)
  }

  stopPolling() {
    clearInterval(this.pollTimer)
  }

  refresh() {
    // Fetch fresh data
  }
}
```


## Resources

- [Stimulus Lifecycle Callbacks](https://stimulus.hotwired.dev/reference/lifecycle-callbacks) — Official reference for initialize, connect, and disconnect callbacks

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*