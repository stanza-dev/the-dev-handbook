---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-setup"
---

# Stimulus Setup

Import Maps provides seamless integration with Stimulus for organizing controllers.

## Controller Setup

```ruby
# config/importmap.rb
pin '@hotwired/stimulus', to: 'stimulus.min.js', preload: true
pin '@hotwired/stimulus-loading', to: 'stimulus-loading.js', preload: true
pin_all_from 'app/javascript/controllers', under: 'controllers'
```

## Controller Index

```javascript
// app/javascript/controllers/index.js
import { application } from 'controllers/application'

import HelloController from 'controllers/hello_controller'
application.register('hello', HelloController)

// Auto-load all controllers (Rails default)
import { eagerLoadControllersFrom } from '@hotwired/stimulus-loading'
eagerLoadControllersFrom('controllers', application)
```

## Creating Controllers

```bash
# Generate a new controller
bin/rails generate stimulus search
```

```javascript
// app/javascript/controllers/search_controller.js
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  static targets = ['input', 'results']
  
  search() {
    const query = this.inputTarget.value
    // Search logic
  }
}
```

## Lazy Loading Controllers

For large applications, lazy load less-used controllers:

```javascript
// app/javascript/controllers/index.js
import { lazyLoadControllersFrom } from '@hotwired/stimulus-loading'

// Eager load common controllers
eagerLoadControllersFrom('controllers', application)

// Lazy load admin controllers
lazyLoadControllersFrom('controllers/admin', application)
```

## Namespaced Controllers

```
app/javascript/controllers/
├── application.js
├── index.js
├── search_controller.js
└── admin/
    ├── users_controller.js
    └── dashboard_controller.js
```

```html
<!-- Use with double-dash for namespaced controllers -->
<div data-controller="admin--users">
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*