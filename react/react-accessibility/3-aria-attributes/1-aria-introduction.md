---
source_course: "react-accessibility"
source_lesson: "react-accessibility-aria-introduction"
---

# Introduction to ARIA

ARIA (Accessible Rich Internet Applications) adds semantic information to HTML.

## The First Rule of ARIA

> Don't use ARIA if you can use native HTML.

```jsx
// ❌ Bad: Using ARIA when HTML works
<div role="button" tabIndex={0} onClick={handleClick}>
  Submit
</div>

// ✅ Good: Native HTML
<button onClick={handleClick}>Submit</button>
```

## When to Use ARIA

1. Custom widgets (tabs, modals, dropdowns)
2. Dynamic content updates
3. Enhanced descriptions
4. Relationships between elements

## Three Types of ARIA Attributes

### 1. Roles
Define what an element IS:
```jsx
<div role="alert">Error message</div>
<ul role="tablist">...</ul>
<div role="dialog">...</div>
```

### 2. Properties
Define characteristics (don't change often):
```jsx
<input aria-required="true" />
<div aria-labelledby="title-id" />
<button aria-haspopup="menu" />
```

### 3. States
Define current state (changes dynamically):
```jsx
<button aria-pressed="true" />  {/* Toggle button */}
<div aria-expanded="false" />   {/* Expandable section */}
<input aria-invalid="true" />   {/* Validation state */}
```

## Common ARIA Attributes

| Attribute | Purpose |
|-----------|----------|
| `aria-label` | Provides accessible name |
| `aria-labelledby` | References element with name |
| `aria-describedby` | References element with description |
| `aria-hidden` | Hides from accessibility tree |
| `aria-live` | Announces dynamic content |
| `aria-expanded` | Expanded/collapsed state |
| `aria-selected` | Selection state |
| `aria-disabled` | Disabled state |
| `aria-invalid` | Validation state |
| `aria-required` | Required field |

## ARIA Doesn't Change Behavior

```jsx
// ❌ ARIA doesn't make this focusable or handle keyboard
<div role="button">Click</div>

// ✅ You must add ALL the behavior
<div
  role="button"
  tabIndex={0}
  onClick={handleClick}
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      handleClick();
    }
  }}
>
  Click
</div>

// ✅✅ Or just use a button!
<button onClick={handleClick}>Click</button>
```

---

> 📘 *This lesson is part of the [React Accessibility (a11y)](https://stanza.dev/courses/react-accessibility) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*