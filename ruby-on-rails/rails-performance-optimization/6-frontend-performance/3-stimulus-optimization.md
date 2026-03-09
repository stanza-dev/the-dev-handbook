---
source_course: "rails-performance-optimization"
source_lesson: "rails-performance-stimulus-optimization"
---

# Stimulus Performance Patterns

## Introduction
Stimulus is Rails' companion JavaScript framework from Hotwire. It connects HTML elements to JavaScript controllers using data attributes. Writing performant Stimulus code means minimizing DOM queries, debouncing user input, and leveraging Turbo for server-rendered updates.

## Key Concepts
- **Stimulus Controller**: A JavaScript class that connects to DOM elements via `data-controller` attributes.
- **Targets**: Named element references that avoid repeated DOM queries — similar to React refs.
- **Debouncing**: Delaying execution of an action until the user stops typing, preventing excessive requests.

## Real World Context
An instant search feature without debouncing fires an API request on every keystroke — 20 characters = 20 requests. Adding a 300ms debounce reduces it to 2-3 requests, dramatically reducing server load and improving perceived speed.

## Deep Dive

### Debounced Search with Turbo

```javascript
// app/javascript/controllers/search_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static targets = ['input', 'results']

  connect() {
    this.timeout = null
  }

  search() {
    clearTimeout(this.timeout)
    this.timeout = setTimeout(() => {
      const query = this.inputTarget.value
      if (query.length >= 2) {
        this.resultsTarget.src = `/search?q=${encodeURIComponent(query)}`
      }
    }, 300) // 300ms debounce
  }

  disconnect() {
    clearTimeout(this.timeout)
  }
}
```

```erb
<div data-controller="search">
  <input type="text"
         data-search-target="input"
         data-action="input->search#search"
         placeholder="Search products...">

  <%= turbo_frame_tag 'search-results',
        data: { search_target: 'results' } do %>
    <!-- Results load here -->
  <% end %>
</div>
```

### Efficient DOM Access with Targets

```javascript
// BAD: Queries DOM on every call
action() {
  const element = this.element.querySelector('[data-role="output"]')
  element.textContent = 'Updated'
}

// GOOD: Use targets (cached reference)
static targets = ['output']

action() {
  this.outputTarget.textContent = 'Updated'
}
```

### Lazy Initialization

```javascript
export default class extends Controller {
  static targets = ['chart']

  // Only initialize chart library when it becomes visible
  chartTargetConnected(element) {
    this.chart = new Chart(element, this.chartConfig)
  }

  chartTargetDisconnected() {
    this.chart?.destroy()
  }
}
```

## Common Pitfalls
1. **Not debouncing input events** — `input` fires on every keystroke. Without debouncing, you create excessive server requests.
2. **Querying the DOM instead of using targets** — `querySelector` in action handlers runs on every invocation. Targets are resolved once and cached.

## Best Practices
1. **Clean up in disconnect()** — Clear timeouts, destroy libraries, and remove event listeners to prevent memory leaks during Turbo navigation.
2. **Use targetConnected/targetDisconnected** — These lifecycle callbacks handle elements appearing and disappearing, perfect for lazy initialization.

## Summary
- Debounce input events to prevent excessive server requests.
- Use Stimulus targets instead of querySelector for efficient DOM access.
- Clean up resources in `disconnect()` to prevent memory leaks.
- Use `targetConnected` for lazy initialization of heavy libraries.

## Code Examples

**Debounced search controller — waits 300ms after the user stops typing before fetching results via Turbo Frame**

```javascript
// Debounced search controller
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static targets = ['input', 'results']

  search() {
    clearTimeout(this.timeout)
    this.timeout = setTimeout(() => {
      const query = this.inputTarget.value
      this.resultsTarget.src = `/search?q=${encodeURIComponent(query)}`
    }, 300)
  }

  disconnect() {
    clearTimeout(this.timeout)
  }
}
```


## Resources

- [Stimulus Handbook](https://stimulus.hotwired.dev/handbook/introduction) — Official Stimulus documentation covering controllers, targets, and actions

---

> 📘 *This lesson is part of the [Rails Performance Optimization](https://stanza.dev/courses/rails-performance-optimization) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*