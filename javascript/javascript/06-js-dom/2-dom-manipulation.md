---
source_course: "javascript"
source_lesson: "javascript-dom-manipulation"
---

# Manipulating DOM Elements

## Introduction

Once you've selected elements, you can modify them—change content, add classes, update attributes, and create new elements. These operations power every dynamic web interface.

## Key Concepts

**innerHTML/textContent**: Properties for getting/setting element content.

**classList**: API for managing CSS classes.

**Attributes**: HTML attributes like `id`, `href`, `data-*`.

## Real World Context

Updating shopping cart counts, showing error messages, toggling dark mode, building dynamic forms—all require DOM manipulation. Understanding these APIs is essential for frontend development.

## Deep Dive

### Modifying Content

```javascript
const elem = document.querySelector('#content');

// Text content (safe, no HTML parsing)
elem.textContent = 'Hello World';
elem.textContent = '<b>Safe</b>';  // Shows literal '<b>Safe</b>'

// HTML content (parses HTML - be careful!)
elem.innerHTML = '<b>Bold</b> text';  // Renders bold
// WARNING: Never use innerHTML with user input (XSS risk)

// Outer HTML (includes the element itself)
console.log(elem.outerHTML);  // '<div id="content">...</div>'
```

### Managing Classes

```javascript
const elem = document.querySelector('.box');

// classList API
elem.classList.add('active');        // Add class
elem.classList.remove('active');     // Remove class
elem.classList.toggle('active');     // Toggle on/off
elem.classList.contains('active');   // Check (returns boolean)
elem.classList.replace('old', 'new'); // Replace

// Add multiple classes
elem.classList.add('one', 'two', 'three');
```

### Working with Attributes

```javascript
const link = document.querySelector('a');

// Get/set attributes
link.getAttribute('href');
link.setAttribute('href', 'https://example.com');
link.hasAttribute('target');
link.removeAttribute('target');

// Data attributes
// <div data-user-id="123" data-role="admin">
const elem = document.querySelector('[data-user-id]');
elem.dataset.userId;  // '123' (camelCase!)
elem.dataset.role;    // 'admin'
elem.dataset.newProp = 'value';  // Sets data-new-prop
```

### Modifying Styles

```javascript
const elem = document.querySelector('.box');

// Inline styles
elem.style.backgroundColor = 'red';  // camelCase
elem.style.fontSize = '16px';
elem.style.cssText = 'color: blue; padding: 10px;';

// Get computed styles
const styles = getComputedStyle(elem);
styles.backgroundColor;  // 'rgb(255, 0, 0)'
```

### Creating and Inserting Elements

```javascript
// Create element
const div = document.createElement('div');
div.className = 'card';
div.textContent = 'New card';

// Insert into DOM
parent.appendChild(div);           // Add as last child
parent.prepend(div);               // Add as first child
parent.insertBefore(div, sibling); // Before specific element

// Modern insertion methods
elem.before(newElem);    // Insert before elem
elem.after(newElem);     // Insert after elem
elem.replaceWith(newElem); // Replace elem

// Insert HTML strings
elem.insertAdjacentHTML('beforeend', '<span>HTML</span>');
// Positions: 'beforebegin', 'afterbegin', 'beforeend', 'afterend'
```

### Removing Elements

```javascript
const elem = document.querySelector('.to-remove');

// Modern way
elem.remove();

// Legacy way
elem.parentElement.removeChild(elem);
```

## Common Pitfalls

1. **XSS with innerHTML**: Never insert untrusted content with innerHTML.
2. **Style property names**: Use camelCase (`backgroundColor`, not `background-color`).
3. **Forgetting units**: `elem.style.width = 100` doesn't work; use `'100px'`.

## Best Practices

- **Use textContent over innerHTML**: When inserting plain text.
- **Prefer classList over className**: More flexible, doesn't overwrite.
- **Use data attributes for custom data**: Not custom attributes.
- **Batch DOM changes**: Modify elements before inserting into DOM.

## Summary

Modify content with `textContent` (safe) or `innerHTML` (HTML parsing). Use `classList` for class management. Access data attributes via `dataset`. Create elements with `createElement()` and insert with `appendChild()` or modern methods like `before()`/`after()`.

## Resources

- [MDN: Element.classList](https://developer.mozilla.org/en-US/docs/Web/API/Element/classList) — classList API reference
- [MDN: Element.innerHTML](https://developer.mozilla.org/en-US/docs/Web/API/Element/innerHTML) — innerHTML property reference

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*