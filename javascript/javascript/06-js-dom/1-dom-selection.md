---
source_course: "javascript"
source_lesson: "javascript-dom-selection"
---

# Selecting DOM Elements

## Introduction

The Document Object Model (DOM) represents your HTML as a tree of objects that JavaScript can manipulate. Before you can change anything, you need to select elements. Modern JavaScript provides powerful selection methods that use CSS selectors.

## Key Concepts

**DOM**: A programming interface that represents HTML/XML as a tree of objects.

**Node**: Any item in the DOM tree (elements, text, comments).

**Element**: A specific type of node representing HTML tags.

## Real World Context

Every interactive feature requires DOM selection: form validation, dynamic content updates, animations, and single-page applications. Efficient selection is the foundation of performant web applications.

## Deep Dive

### querySelector and querySelectorAll

```javascript
// Select first matching element
const header = document.querySelector('#header');
const firstButton = document.querySelector('button');
const active = document.querySelector('.active');

// Select all matching elements (returns NodeList)
const allButtons = document.querySelectorAll('button');
const items = document.querySelectorAll('.list-item');

// Complex selectors
const nested = document.querySelector('nav > ul > li:first-child');
const dataAttr = document.querySelector('[data-id="123"]');
```

### Legacy Methods (Still Useful)

```javascript
// By ID (faster than querySelector for IDs)
const elem = document.getElementById('myId');

// By class (returns live HTMLCollection)
const items = document.getElementsByClassName('item');

// By tag (returns live HTMLCollection)
const divs = document.getElementsByTagName('div');
```

### NodeList vs HTMLCollection

```javascript
// querySelectorAll returns static NodeList
const nodeList = document.querySelectorAll('.item');
nodeList.forEach(item => console.log(item));  // forEach works!

// getElementsByClassName returns live HTMLCollection
const htmlCollection = document.getElementsByClassName('item');
// htmlCollection.forEach(...)  // Error! No forEach
Array.from(htmlCollection).forEach(item => console.log(item));
```

### Scoped Selection

```javascript
const container = document.querySelector('.container');

// Search within container only
const innerButton = container.querySelector('button');
const innerItems = container.querySelectorAll('.item');
```

### Traversing the DOM

```javascript
const elem = document.querySelector('.target');

// Parent/children
elem.parentElement;
elem.children;  // HTMLCollection of child elements
elem.firstElementChild;
elem.lastElementChild;

// Siblings
elem.previousElementSibling;
elem.nextElementSibling;

// Closest ancestor matching selector
elem.closest('.container');
```

## Common Pitfalls

1. **querySelector returns null if not found**: Always check before using.
2. **Live vs static collections**: `getElementsByClassName` updates automatically; `querySelectorAll` doesn't.
3. **Forgetting the `#` or `.`**: `querySelector('header')` selects `<header>` tag, not id.

## Best Practices

- **Prefer `querySelector`/`querySelectorAll`**: More flexible, familiar CSS syntax.
- **Use `getElementById` for IDs**: Slightly faster when you need just one element by ID.
- **Cache selections**: Don't re-query the same elements repeatedly.
- **Check for null**: `querySelector` returns null if nothing matches.

## Summary

`querySelector` and `querySelectorAll` select elements using CSS selectors. Use `#` for IDs, `.` for classes. `querySelectorAll` returns a static NodeList; legacy methods return live HTMLCollections. Use `closest()` to find ancestors and cache selections for performance.

## Resources

- [MDN: Document.querySelector()](https://developer.mozilla.org/en-US/docs/Web/API/Document/querySelector) — querySelector reference
- [MDN: Introduction to the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Introduction) — DOM concepts overview

---

> 📘 *This lesson is part of the [JavaScript Core Mastery](https://stanza.dev/courses/javascript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*