---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-lifecycle"
---

# Targets and Values

## Introduction
Targets provide named references to DOM elements within a controller's scope. Values store typed data on the controller element via data attributes. Together, they eliminate the need for manual DOM queries and data parsing.

## Key Concepts
- **Targets**: Named DOM element references declared with `static targets` and accessed via `this.nameTarget`.
- **Values**: Typed data attributes declared with `static values` and accessed via `this.nameValue`.
- **Target Callbacks**: Methods called when targets connect or disconnect (e.g., `nameTargetConnected`).
- **Value Changed Callbacks**: Methods called when a value attribute changes (e.g., `nameValueChanged`).

## Real World Context
Instead of littering your controller with `document.querySelector` calls and `JSON.parse` on data attributes, targets and values provide a clean, declarative API. They make controllers self-documenting — you can see at a glance what DOM elements and configuration a controller needs.

## Deep Dive
### Declaring and Using Targets

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "results", "count"]

  search() {
    const query = this.inputTarget.value
    // this.inputTarget — single element (throws if missing)
    // this.inputTargets — array of all matching elements
    // this.hasInputTarget — boolean check
    this.resultsTarget.innerHTML = `Searching: ${query}`
    this.countTarget.textContent = this.resultsTargets.length
  }
}
```

In HTML, targets are marked with `data-[controller]-target`:

```html
<div data-controller="search">
  <input data-search-target="input" data-action="input->search#search">
  <span data-search-target="count">0</span>
  <div data-search-target="results"></div>
</div>
```

Each target name generates three properties: `nameTarget` (first match), `nameTargets` (all matches), and `hasNameTarget` (boolean).

### Declaring and Using Values

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static values = {
    url: String,
    refreshInterval: { type: Number, default: 5000 },
    autoStart: { type: Boolean, default: false }
  }

  connect() {
    if (this.autoStartValue) {
      this.startRefreshing()
    }
  }

  startRefreshing() {
    setInterval(() => {
      fetch(this.urlValue)
        .then(r => r.text())
        .then(html => this.element.innerHTML = html)
    }, this.refreshIntervalValue)
  }
}
```

Values are passed via data attributes in HTML:

```html
<div data-controller="dashboard"
     data-dashboard-url-value="/api/stats"
     data-dashboard-refresh-interval-value="10000"
     data-dashboard-auto-start-value="true">
</div>
```

Supported types: String, Number, Boolean, Array, Object. Stimulus handles parsing automatically.

### Value Changed Callbacks

```javascript
static values = { count: { type: Number, default: 0 } }

countValueChanged(newValue, oldValue) {
  // Called whenever data-controller-count-value changes
  this.element.querySelector(".badge").textContent = newValue
}
```

These callbacks fire whenever the data attribute is updated, even from external code. They enable reactive patterns without a framework.

## Common Pitfalls
1. **Accessing `nameTarget` when it doesn't exist** — Throws an error. Use `hasNameTarget` to check first, or use `nameTargets` (empty array if none).
2. **Wrong data attribute format** — Values use `data-[controller]-[name]-value`. The kebab-case attribute maps to camelCase in JavaScript.

## Best Practices
1. **Use values for configuration, targets for DOM** — Values hold data (URLs, intervals, flags). Targets reference elements.
2. **Prefer value changed callbacks over observers** — They are cleaner than MutationObserver for reacting to data changes.

## Summary
- Targets provide named DOM references via `static targets` and `data-*-target` attributes.
- Values store typed configuration via `static values` and `data-*-value` attributes.
- Three accessor patterns: `nameTarget` (single), `nameTargets` (array), `hasNameTarget` (boolean).
- Value changed callbacks fire automatically when data attributes change.
- Supported value types: String, Number, Boolean, Array, Object.

## Code Examples

**A controller combining targets (DOM references) and values (typed configuration with change callbacks)**

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["output"]
  static values = {
    greeting: { type: String, default: "Hello" },
    count: { type: Number, default: 0 }
  }

  greet() {
    this.countValue++  // Triggers countValueChanged
    this.outputTarget.textContent =
      `${this.greetingValue}! (clicked ${this.countValue} times)`
  }

  countValueChanged() {
    // Automatically called when countValue changes
    console.log(`Count is now: ${this.countValue}`)
  }
}
```


## Resources

- [Stimulus Values](https://stimulus.hotwired.dev/reference/values) — Official reference for declaring and using values with type coercion

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*