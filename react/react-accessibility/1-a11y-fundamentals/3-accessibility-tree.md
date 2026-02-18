---
source_course: "react-accessibility"
source_lesson: "react-accessibility-accessibility-tree"
---

# The Accessibility Tree

Browsers create an accessibility tree from your HTML that assistive technologies use.

## DOM vs Accessibility Tree

```html
<!-- DOM -->
<nav>
  <ul>
    <li><a href="/">Home</a></li>
    <li><a href="/about">About</a></li>
  </ul>
</nav>

<!-- Accessibility Tree (simplified) -->
navigation
  list
    listitem
      link "Home"
    listitem
      link "About"
```

## How Elements Map

| HTML Element | Accessibility Role |
|-------------|--------------------|
| `<button>` | button |
| `<a href>` | link |
| `<input type="text">` | textbox |
| `<input type="checkbox">` | checkbox |
| `<nav>` | navigation |
| `<main>` | main |
| `<header>` | banner |
| `<footer>` | contentinfo |
| `<h1>` | heading level 1 |
| `<ul>` | list |
| `<li>` | listitem |

## Inspecting the Accessibility Tree

### Chrome DevTools

1. Open DevTools (F12)
2. Elements panel
3. Click "Accessibility" tab
4. See computed accessibility properties

### Firefox

1. Open DevTools
2. Click "Accessibility" tab
3. View full accessibility tree

## What Gets Exposed

```jsx
// Role, Name, State
<button disabled>Submit</button>

// Accessibility properties:
// Role: button
// Name: "Submit" (from text content)
// State: disabled
```

## Common Issues

```jsx
// ❌ Div with click handler - no role, not focusable
<div onClick={handleClick}>Submit</div>

// ✅ Button - proper role, focusable, keyboard support
<button onClick={handleClick}>Submit</button>

// ❌ Image without alt - no accessible name
<img src="chart.png" />

// ✅ Image with alt - has accessible name
<img src="chart.png" alt="Sales increased 50% in Q4" />
```

## React and the Accessibility Tree

React renders to the DOM, which creates the accessibility tree. Your JSX choices directly impact accessibility:

```jsx
// Your JSX
<main>
  <h1>Dashboard</h1>
  <button onClick={logout}>Log out</button>
</main>

// Creates accessibility tree:
// main
//   heading "Dashboard" level=1
//   button "Log out"
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*