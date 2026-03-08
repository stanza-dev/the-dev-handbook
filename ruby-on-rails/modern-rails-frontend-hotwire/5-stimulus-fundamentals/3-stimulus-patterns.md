---
source_course: "modern-rails-frontend-hotwire"
source_lesson: "rails-hotwire-stimulus-patterns"
---

# Actions and Events

## Introduction
Stimulus actions map DOM events to controller methods using `data-action` attributes. Instead of writing `addEventListener` calls, you declare the event binding directly in HTML, keeping behavior close to the markup it affects.

## Key Concepts
- **Action Descriptor**: The `data-action` attribute format: `event->controller#method`.
- **Default Events**: Some elements have default events (click for buttons, submit for forms, input for inputs).
- **Event Options**: Modifiers like `:prevent`, `:stop`, `:once` appended to the action descriptor.
- **Action Parameters**: Passing data from HTML to action methods via `data-[controller]-[param]-param`.

## Real World Context
Actions are how you connect user interactions to behavior. Clicking a button, submitting a form, pressing a key, hovering an element — every event-driven interaction in a Stimulus-powered app uses the action descriptor syntax.

## Deep Dive
### Action Descriptor Syntax

The full format is `event->controller#method`:

```html
<div data-controller="clipboard">
  <input data-clipboard-target="source" value="Copy me!">
  <button data-action="click->clipboard#copy">Copy</button>
</div>
```

The button's click event triggers the `copy` method on the `clipboard` controller. The format always follows: `event->controller#method`.

### Default Events

Some elements have default events that can be omitted:

```html
<!-- These are equivalent -->
<button data-action="click->modal#open">Open</button>
<button data-action="modal#open">Open</button>  <!-- click is default for button -->

<form data-action="submit->search#perform">  <!-- explicit -->
<form data-action="search#perform">           <!-- submit is default for form -->

<input data-action="input->filter#update">    <!-- explicit -->
<input data-action="filter#update">           <!-- input is default for input -->
```

Defaults: `click` for links/buttons, `submit` for forms, `input` for input/textarea/select.

### Event Modifiers

Append modifiers with colons:

```html
<!-- Prevent default behavior -->
<form data-action="submit->form#save:prevent">

<!-- Stop event propagation -->
<button data-action="click->menu#toggle:stop">

<!-- Fire only once -->
<button data-action="click->intro#dismiss:once">

<!-- Keyboard-specific: only on Enter key -->
<input data-action="keydown.enter->search#perform">

<!-- Combine modifiers -->
<a data-action="click->modal#open:prevent:stop">
```

Modifiers eliminate the need for `event.preventDefault()` and `event.stopPropagation()` in your JavaScript.

### Action Parameters

Pass data from HTML to action methods:

```html
<div data-controller="cart">
  <button data-action="click->cart#add"
          data-cart-product-id-param="42"
          data-cart-quantity-param="1">
    Add to Cart
  </button>
</div>
```

Access parameters in the action method:

```javascript
add(event) {
  const { productId, quantity } = event.params
  console.log(`Adding product ${productId}, qty ${quantity}`)
  // productId is "42", quantity is "1"
}
```

Parameters are automatically extracted from `data-[controller]-[name]-param` attributes on the element that triggered the action.

### Multiple Actions on One Element

Space-separate multiple actions:

```html
<input data-action="input->search#update focus->search#expand blur->search#collapse">
```

This element responds to three different events, each calling a different method on the same controller.

## Common Pitfalls
1. **Wrong action format** — The delimiter is `->` (not `.` or `:`). The format is `event->controller#method`.
2. **Forgetting the controller name** — `data-action="click->copy"` is wrong. It must include both controller and method: `click->clipboard#copy`.

## Best Practices
1. **Use modifiers instead of JavaScript event methods** — `:prevent` and `:stop` are cleaner than calling `event.preventDefault()` in your method.
2. **Use action parameters for per-element data** — Instead of querying data attributes manually, use the `data-*-param` pattern for clean data passing.

## Summary
- Actions use the format `event->controller#method` in `data-action` attributes.
- Default events (click, submit, input) can be omitted for common elements.
- Modifiers (`:prevent`, `:stop`, `:once`, `.enter`) control event behavior declaratively.
- Action parameters pass data from HTML to methods via `data-*-param` attributes.
- Multiple actions can be space-separated on a single element.

## Code Examples

**A validation controller using action parameters to receive min/max constraints from HTML attributes**

```javascript
import { Controller } from "@hotwired/stimulus"

export default class extends Controller {
  static targets = ["field", "output"]

  // Action with event parameter
  validate({ target, params }) {
    const value = target.value
    const { min, max } = params  // from data-validator-min-param, data-validator-max-param

    if (value.length < min || value.length > max) {
      this.outputTarget.textContent = `Must be ${min}-${max} characters`
      this.outputTarget.classList.add("text-red-500")
    } else {
      this.outputTarget.textContent = "Looks good!"
      this.outputTarget.classList.remove("text-red-500")
    }
  }
}

// <div data-controller="validator">
//   <input data-action="input->validator#validate"
//          data-validator-min-param="3"
//          data-validator-max-param="50">
//   <p data-validator-target="output"></p>
// </div>
```


## Resources

- [Stimulus Actions](https://stimulus.hotwired.dev/reference/actions) — Official reference for action descriptors, events, and parameters

---

> 📘 *This lesson is part of the [Modern Rails Frontend with Hotwire](https://stanza.dev/courses/modern-rails-frontend-hotwire) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*