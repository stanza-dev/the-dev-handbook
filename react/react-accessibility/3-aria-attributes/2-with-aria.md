---
source_course: "react-accessibility"
source_lesson: "react-accessibility-labeling-with-aria"
---

# Labeling Elements with ARIA

Every interactive element needs an accessible name.

## aria-label

Provide an accessible name directly:

```jsx
// Icon-only button
<button aria-label="Close">
  <CloseIcon />
</button>

// Search input without visible label
<input
  type="search"
  aria-label="Search products"
  placeholder="Search..."
/>

// Navigation with aria-label
<nav aria-label="Main navigation">
  {/* links */}
</nav>
<nav aria-label="Footer navigation">
  {/* links */}
</nav>
```

## aria-labelledby

Reference another element's text:

```jsx
// Dialog with title
<div
  role="dialog"
  aria-labelledby="dialog-title"
  aria-describedby="dialog-desc"
>
  <h2 id="dialog-title">Confirm Delete</h2>
  <p id="dialog-desc">This action cannot be undone.</p>
  {/* ... */}
</div>

// Multiple labels
<button aria-labelledby="item-name item-price">
  <span id="item-name">Widget</span>
  <span id="item-price">$9.99</span>
</button>
// Announced as: "Widget $9.99, button"
```

## aria-describedby

Provide additional description:

```jsx
<label htmlFor="password">Password</label>
<input
  id="password"
  type="password"
  aria-describedby="password-requirements password-error"
/>
<p id="password-requirements">
  Must be at least 8 characters
</p>
{error && (
  <p id="password-error" role="alert">
    {error}
  </p>
)}
```

## Label Priority

How accessible name is determined (in order):

1. `aria-labelledby`
2. `aria-label`
3. `<label>` element
4. `title` attribute
5. Text content

```jsx
// aria-labelledby wins
<button
  aria-label="Cancel" // Ignored
  aria-labelledby="custom-label" // Used!
>
  Delete {/* Also ignored */}
</button>
<span id="custom-label">Close dialog</span>
// Announced as: "Close dialog, button"
```

## visually-hidden Class

For screen-reader-only content:

```css
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

```jsx
<button>
  <TrashIcon aria-hidden="true" />
  <span className="sr-only">Delete item</span>
</button>
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*