---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-lifecycle"
---

# Stimulus Lifecycle

Stimulus controllers have lifecycle callbacks and powerful event handling.

## Lifecycle Callbacks

```javascript
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  // Called once when controller connects
  connect() {
    console.log('Controller connected')
  }
  
  // Called when controller disconnects
  disconnect() {
    console.log('Controller disconnected')
    this.cleanup()
  }
  
  // Called when target connects (dynamic content)
  itemTargetConnected(element) {
    console.log('New item:', element)
  }
  
  // Called when target disconnects
  itemTargetDisconnected(element) {
    console.log('Item removed:', element)
  }
}
```

## Event Options

```html
<!-- Prevent default -->
<form data-action="submit->form#save:prevent">

<!-- Stop propagation -->
<button data-action="click->modal#close:stop">

<!-- Once only -->
<div data-action="click->tracker#track:once">

<!-- Passive listener -->
<div data-action="scroll->scroller#onScroll:passive">
```

## Global Events

```html
<!-- Window events -->
<div data-controller="keyboard" data-action="keydown@window->keyboard#handle">

<!-- Document events -->
<div data-action="turbo:load@document->analytics#pageView">
```

## Custom Events

```javascript
// Dispatching custom events
import { Controller } from '@hotwired/stimulus'

export default class extends Controller {
  select(event) {
    const item = event.currentTarget
    
    // Dispatch custom event
    this.dispatch('selected', {
      detail: { id: item.dataset.id }
    })
  }
}
```

```html
<!-- Listening to custom events -->
<div data-controller="list">
  <div data-controller="item" 
       data-action="item:selected->list#handleSelection">
    <button data-action="click->item#select">Select</button>
  </div>
</div>
```

## Cleaning Up

```javascript
export default class extends Controller {
  connect() {
    this.interval = setInterval(() => this.refresh(), 5000)
    this.observer = new IntersectionObserver(this.onIntersect.bind(this))
  }
  
  disconnect() {
    clearInterval(this.interval)
    this.observer.disconnect()
  }
}
```

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*