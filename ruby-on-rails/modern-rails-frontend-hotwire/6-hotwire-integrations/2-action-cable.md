---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-hotwire-action-cable"
---

# Stimulus and Turbo Integration

## Introduction
Stimulus and Turbo are designed to work together seamlessly. Stimulus controllers enhance Turbo-driven pages with JavaScript behavior that survives page transitions, while Turbo events trigger Stimulus actions.

## Key Concepts
- **Morphing Compatibility**: Stimulus controllers persist through Turbo morph updates because morph preserves DOM elements.
- **Turbo Events**: Custom events like `turbo:submit-start` and `turbo:frame-load` can trigger Stimulus actions.
- **Auto-reconnection**: When Turbo swaps page content, Stimulus automatically disconnects old controllers and connects new ones.

## Real World Context
A real application needs both Turbo and Stimulus working together. Turbo handles navigation and partial updates. Stimulus handles UI behaviors: dropdowns, modals, form validation, animations, and third-party library integration. The key is understanding how they interact during page transitions.

## Deep Dive
### Listening to Turbo Events

Stimulus controllers can react to Turbo lifecycle events:

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  connect() {
    this.element.addEventListener("turbo:submit-start", this.onSubmitStart)
    this.element.addEventListener("turbo:submit-end", this.onSubmitEnd)
  }

  disconnect() {
    this.element.removeEventListener("turbo:submit-start", this.onSubmitStart)
    this.element.removeEventListener("turbo:submit-end", this.onSubmitEnd)
  }

  onSubmitStart = () => {
    this.element.classList.add("opacity-50")
  }

  onSubmitEnd = () => {
    this.element.classList.remove("opacity-50")
  }
}
```

This controller dims a form during Turbo submission and restores it when complete. The Turbo events bubble, so the controller catches them on its element.

### Frame Load Events

```javascript
// React when a Turbo Frame finishes loading
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  handleFrameLoad() {
    // Re-initialize anything after frame content is swapped
    this.element.querySelectorAll("[data-animate]").forEach(el => {
      el.classList.add("animate-fade-in")
    })
  }
}
```

```html
<turbo-frame id="results"
             data-controller="frame-handler"
             data-action="turbo:frame-load->frame-handler#handleFrameLoad">
</turbo-frame>
```

The `turbo:frame-load` event fires after a frame's content is swapped, letting Stimulus run post-load logic.

### Triggering Turbo Frame Reloads from Stimulus

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = { interval: Number }

  connect() {
    this.timer = setInterval(() => {
      const frame = document.querySelector("turbo-frame#notifications")
      if (frame && frame.src) {
        frame.reload()
      }
    }, this.intervalValue)
  }

  disconnect() {
    clearInterval(this.timer)
  }
}
```

Stimulus can programmatically reload Turbo Frames using the `reload()` method. This enables auto-refreshing sections without full-page polling.

## Common Pitfalls
1. **Manually querying DOM after Turbo swap** — After Turbo replaces content, old DOM references are stale. Use Stimulus targets instead of cached querySelector results.
2. **Event listeners without cleanup** — If you add event listeners in `connect()`, always remove them in `disconnect()`. Turbo's page transitions mean controllers connect/disconnect frequently.

## Best Practices
1. **Use data-action for Turbo events** — Instead of manual addEventListener, use `data-action="turbo:submit-start->form#disable"` for cleaner code.
2. **Let Turbo handle data fetching** — Use frames and streams for loading data from the server. Reserve Stimulus for client-side behavior that doesn't need server interaction.

## Summary
- Stimulus controllers survive Turbo morph updates and reconnect automatically after page swaps.
- Turbo custom events (submit-start, submit-end, frame-load) can trigger Stimulus actions.
- Stimulus can programmatically reload Turbo Frames for auto-refreshing patterns.
- Always clean up event listeners in disconnect() to prevent leaks during Turbo navigation.
- Use data-action for Turbo events instead of manual addEventListener.

## Code Examples

**A Stimulus controller that hooks into Turbo form submission events to show/hide a loading spinner**

```javascript
// A form controller that integrates with Turbo submission events
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["submit", "spinner"]

  disableOnSubmit() {
    this.submitTarget.disabled = true
    this.spinnerTarget.classList.remove("hidden")
  }

  enableAfterSubmit() {
    this.submitTarget.disabled = false
    this.spinnerTarget.classList.add("hidden")
  }
}

// <form data-controller="form-status"
//       data-action="turbo:submit-start->form-status#disableOnSubmit
//                    turbo:submit-end->form-status#enableAfterSubmit">
//   <button data-form-status-target="submit">Save</button>
//   <span data-form-status-target="spinner" class="hidden">⏳</span>
// </form>
```


## Resources

- [Turbo Reference Events](https://turbo.hotwired.dev/reference/events) — Complete list of Turbo events that can be used with Stimulus

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*