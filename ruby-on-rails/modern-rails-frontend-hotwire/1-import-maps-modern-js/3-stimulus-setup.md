---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-setup"
---

# Stimulus Setup with Import Maps

## Introduction
Stimulus is the JavaScript framework in Hotwire, designed to add behavior to HTML through controllers. Rails 8 ships Stimulus fully configured through Import Maps — no additional installation needed.

## Key Concepts
- **Stimulus Controller**: A JavaScript class connected to HTML elements via `data-controller` attributes.
- **Targets**: Named references to DOM elements declared with `static targets`.
- **Actions**: Event handlers mapped to methods via `data-action` attributes.
- **Auto-registration**: `eagerLoadControllersFrom` automatically registers all `_controller.js` files.

## Real World Context
You will use Stimulus controllers for every interactive behavior: toggling menus, form validation, showing/hiding elements, modals, and integrating third-party libraries. Unlike SPA frameworks, Stimulus enhances server-rendered HTML rather than replacing it.

## Deep Dive
### Default File Structure

Rails 8 generates Stimulus setup automatically:

```
app/javascript/
├── application.js
└── controllers/
    ├── application.js
    └── hello_controller.js
```

The shared application instance:

```javascript
// app/javascript/controllers/application.js
import { Application } from "@hotwired/stimulus"
const application = Application.start()
application.debug = false
window.Stimulus = application
export { application }
```

This creates a single Stimulus Application instance. `window.Stimulus` helps debugging in the console.

### Auto-registration

```javascript
// app/javascript/controllers/index.js
import { application } from "controllers/application"
import { eagerLoadControllersFrom } from "@hotwired/stimulus-loading"
eagerLoadControllersFrom("controllers", application)
```

`eagerLoadControllersFrom` scans the `controllers` directory and registers every `_controller.js` file. Just create the file and it works.

### Creating Controllers

```bash
bin/rails generate stimulus search
```

This generates:

```javascript
// app/javascript/controllers/search_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["input", "results"]

  search() {
    const query = this.inputTarget.value
    this.resultsTarget.textContent = `Searching for: ${query}`
  }
}
```

Targets become properties like `this.inputTarget`. Connect HTML with data attributes:

```html
<div data-controller="search">
  <input data-search-target="input" data-action="input->search#search">
  <p data-search-target="results"></p>
</div>
```

`data-controller` instantiates the class. `data-action` maps events to methods. `data-*-target` creates named references.

### Namespaced Controllers

Subdirectories use double-dash notation:

```
controllers/admin/users_controller.js → data-controller="admin--users"
```

## Common Pitfalls
1. **Forgetting `_controller.js` suffix** — Auto-registration depends on this naming convention.
2. **Wrong data-action syntax** — Format is `event->controller#method`. Dots and colons won't work.

## Best Practices
1. **Use the generator** — `bin/rails generate stimulus name` ensures correct naming and boilerplate.
2. **Keep controllers small** — One concern per controller. Split at 50-60 lines.

## Summary
- Rails 8 ships Stimulus configured with Import Maps.
- Controllers in `app/javascript/controllers/` ending with `_controller.js` are auto-registered.
- HTML connects via `data-controller`, `data-*-target`, and `data-action`.
- Namespaced controllers use subdirectories and double-dash notation.

## Code Examples

**A toggle controller showing targets, CSS classes, and actions — the most common Stimulus pattern**

```javascript
// app/javascript/controllers/toggle_controller.js
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["content"]
  static classes = ["hidden"]

  toggle() {
    this.contentTarget.classList.toggle(this.hiddenClass)
  }
}
// <div data-controller="toggle" data-toggle-hidden-class="hidden">
//   <button data-action="click->toggle#toggle">Toggle</button>
//   <div data-toggle-target="content">Content here</div>
// </div>
```


## Resources

- [Stimulus Handbook](https://stimulus.hotwired.dev/handbook/introduction) — Official Stimulus handbook covering controllers, actions, targets, and values

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*