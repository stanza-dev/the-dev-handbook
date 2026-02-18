---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-patterns"
---

# Stimulus Patterns

Learn patterns for building reusable Stimulus controllers.

## Outlets (Controller Communication)

```javascript
// controllers/form_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static outlets = ['submit-button']
  
  validate() {
    const isValid = this.checkValidity()
    
    this.submitButtonOutlets.forEach(button => {
      button.toggle(isValid)
    })
  }
}

// controllers/submit_button_controller.js
export default class extends Controller {
  toggle(enabled) {
    this.element.disabled = !enabled
  }
}
```

```html
<form data-controller="form" data-form-submit-button-outlet="#submit">
  <input data-action="input->form#validate">
  <button id="submit" data-controller="submit-button">Submit</button>
</form>
```

## Classes API

```javascript
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static classes = ['active', 'loading']
  
  toggle() {
    this.element.classList.toggle(this.activeClass)
  }
  
  startLoading() {
    this.element.classList.add(this.loadingClass)
  }
}
```

```html
<div data-controller="button"
     data-button-active-class="btn-active"
     data-button-loading-class="btn-loading">
</div>
```

## Async Data Loading

```javascript
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static targets = ['content']
  static values = { url: String }
  
  async connect() {
    await this.load()
  }
  
  async load() {
    this.element.classList.add('loading')
    
    try {
      const response = await fetch(this.urlValue)
      this.contentTarget.innerHTML = await response.text()
    } finally {
      this.element.classList.remove('loading')
    }
  }
}
```

## Debounce Pattern

```javascript
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static values = { delay: { type: Number, default: 300 } }
  
  search() {
    clearTimeout(this.timeout)
    this.timeout = setTimeout(() => {
      this.performSearch()
    }, this.delayValue)
  }
  
  performSearch() {
    // Actual search logic
  }
}
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*