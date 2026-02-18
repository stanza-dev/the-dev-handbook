---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-introduction"
---

# Stimulus Introduction

Stimulus is a modest JavaScript framework for adding behavior to HTML.

## The Stimulus Philosophy

- **HTML-first**: Behavior is added to existing HTML
- **Convention over configuration**: Standard naming patterns
- **Lifecycle aware**: Handles dynamic DOM changes

## Controller Structure

```javascript
// app/javascript/controllers/hello_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  connect() {
    console.log('Hello, Stimulus!')
  }
}
```

```html
<div data-controller="hello">
  <!-- Controller connects when this element appears -->
</div>
```

## Actions

```javascript
// controllers/toggle_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  toggle() {
    this.element.classList.toggle('active')
  }
}
```

```html
<div data-controller="toggle">
  <button data-action="click->toggle#toggle">Toggle</button>
</div>
```

## Targets

```javascript
// controllers/search_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static targets = ['input', 'results']
  
  search() {
    const query = this.inputTarget.value
    this.resultsTarget.innerHTML = `Searching for: ${query}`
  }
}
```

```html
<div data-controller="search">
  <input data-search-target="input" data-action="input->search#search">
  <div data-search-target="results"></div>
</div>
```

## Values

```javascript
// controllers/countdown_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static values = { seconds: Number }
  
  connect() {
    this.start()
  }
  
  start() {
    setInterval(() => {
      this.secondsValue--
    }, 1000)
  }
  
  secondsValueChanged() {
    this.element.textContent = this.secondsValue
  }
}
```

```html
<div data-controller="countdown" data-countdown-seconds-value="60">60</div>
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*