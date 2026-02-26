---
source_course: "javascript"
source_lesson: "javascript-event-propagation"
---

# Event Bubbling & Delegation

## Introduction

Events don't just fire on the target element—they propagate through the DOM tree. Understanding bubbling and capturing enables powerful patterns like event delegation, which can dramatically simplify your code and improve performance.

## Key Concepts

**Bubbling**: Events propagate from target up to ancestors (default).

**Capturing**: Events propagate from ancestors down to target (opt-in).

**Event Delegation**: Handling events on a parent instead of individual children.

## Real World Context

Lists with clickable items, dynamic content, menus with many options—delegation handles all of these efficiently with a single listener instead of hundreds.

## Deep Dive

### Event Propagation Phases

```javascript
// Three phases:
// 1. Capture: window → document → ... → parent → target
// 2. Target: event fires on target
// 3. Bubble: target → parent → ... → document → window

document.body.addEventListener('click', (e) => {
  console.log('Body clicked (bubble phase)');
});

document.body.addEventListener('click', (e) => {
  console.log('Body clicked (capture phase)');
}, true);  // or { capture: true }
```

### Stopping Propagation

```javascript
child.addEventListener('click', (e) => {
  e.stopPropagation();  // Parent won't see this event
  console.log('Child clicked');
});

// stopImmediatePropagation stops other listeners on SAME element too
child.addEventListener('click', (e) => {
  e.stopImmediatePropagation();
});
child.addEventListener('click', (e) => {
  // This won't run!
});
```

### Event Delegation

```javascript
// BAD: Listener on every item
document.querySelectorAll('.item').forEach(item => {
  item.addEventListener('click', handleClick);
});

// GOOD: Single listener on parent
document.querySelector('.list').addEventListener('click', (e) => {
  // Check if clicked element is an item
  if (e.target.matches('.item')) {
    handleItemClick(e.target);
  }
  
  // Or find closest matching ancestor
  const item = e.target.closest('.item');
  if (item) {
    handleItemClick(item);
  }
});
```

### Benefits of Delegation

```javascript
// Works with dynamically added elements!
const list = document.querySelector('.list');

list.addEventListener('click', (e) => {
  if (e.target.matches('.delete-btn')) {
    e.target.closest('.item').remove();
  }
});

// Adding new items - no need to add listeners!
list.insertAdjacentHTML('beforeend', `
  <li class="item">New item <button class="delete-btn">×</button></li>
`);
// The delete button already works!
```

### target vs currentTarget

```javascript
parent.addEventListener('click', (e) => {
  e.target;        // The actual element clicked (could be a child)
  e.currentTarget; // Always the element with the listener (parent)
  
  // Often different when bubbling!
  // Click on child: target=child, currentTarget=parent
});
```

### Practical Example: Tab Component

```javascript
const tabs = document.querySelector('.tabs');

tabs.addEventListener('click', (e) => {
  const tab = e.target.closest('[data-tab]');
  if (!tab) return;
  
  // Remove active from all tabs
  tabs.querySelectorAll('[data-tab]').forEach(t => 
    t.classList.remove('active'));
  
  // Add active to clicked tab
  tab.classList.add('active');
  
  // Show corresponding content
  const contentId = tab.dataset.tab;
  showContent(contentId);
});
```

## Common Pitfalls

1. **Using `e.target` when you mean `e.currentTarget`**: They differ during bubbling.
2. **Overusing `stopPropagation`**: Can break other functionality.
3. **Not using `closest()`**: Direct `e.target` checks miss clicks on child elements.

## Best Practices

- **Use delegation for lists/collections**: More efficient, works with dynamic content.
- **Use `closest()` for delegation**: Handles clicks on nested elements.
- **Avoid `stopPropagation` unless necessary**: Can cause unexpected issues.
- **Prefer delegation over many listeners**: Better performance, less memory.

## Summary

Events bubble up from target to ancestors by default. Event delegation attaches one listener to a parent to handle events on children. Use `e.target.closest(selector)` for robust delegation. `stopPropagation()` stops bubbling but use sparingly. Delegation is essential for dynamic content and better performance.

## Code Examples

**Event Propagation Phases**

```javascript
// Three phases:
// 1. Capture: window → document → ... → parent → target
// 2. Target: event fires on target
// 3. Bubble: target → parent → ... → document → window

document.body.addEventListener('click', (e) => {
  console.log('Body clicked (bubble phase)');
});

document.body.addEventListener('click', (e) => {
  console.log('Body clicked (capture phase)');
}, true);  // or { capture: true }
```

**Stopping Propagation**

```javascript
child.addEventListener('click', (e) => {
  e.stopPropagation();  // Parent won't see this event
  console.log('Child clicked');
});

// stopImmediatePropagation stops other listeners on SAME element too
child.addEventListener('click', (e) => {
  e.stopImmediatePropagation();
});
child.addEventListener('click', (e) => {
  // This won't run!
});
```


## Resources

- [MDN: Event bubbling](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Building_blocks/Event_bubbling) — Understanding event bubbling
- [MDN: Element.closest()](https://developer.mozilla.org/en-US/docs/Web/API/Element/closest) — closest() method for delegation

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*