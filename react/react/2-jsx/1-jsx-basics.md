---
source_course: "react"
source_lesson: "react-jsx-basics"
---

# JSX Basics

## Introduction

JSX is a syntax extension for JavaScript that lets you write HTML-like markup directly inside your component functions. Under the hood, JSX is transformed into regular JavaScript function calls that produce React elements. Mastering JSX rules is essential because virtually every React component you write will use it.

## Key Concepts

- **JSX Element**: A piece of markup like `<div>Hello</div>` that compiles to a `React.createElement` call (or the new JSX transform) at build time.
- **Expressions in Curly Braces**: Any JavaScript expression placed inside `{}` will be evaluated and rendered.
- **Fragments**: The `<></>` syntax lets you return multiple elements without adding an extra DOM node.
- **Self-Closing Tags**: Every tag must be closed. Tags without children use `/>` (e.g., `<img />`, `<br />`).

## Real World Context

When building a user profile card, you need to combine static markup (headings, containers) with dynamic data (user name, avatar URL, bio). JSX makes this natural: you write the HTML structure and drop JavaScript expressions wherever you need dynamic values. Without JSX, you would chain `createElement` calls, which quickly becomes unreadable for complex UIs.

## Deep Dive

JSX follows a few strict rules that differ from HTML:

**1. Return a single root element**

Every component must return one root element. Use a fragment if you do not want an extra DOM wrapper:

```tsx
// Using a fragment to avoid an unnecessary <div>
function UserInfo({ name, email }: { name: string; email: string }) {
  return (
    <>
      <h2>{name}</h2>
      <p>{email}</p>
    </>
  );
}
```

**2. Use camelCase for attributes**

HTML attributes are renamed to camelCase in JSX because JSX is closer to JavaScript than HTML:

```tsx
<div className="card" tabIndex={0} onClick={handleClick}>
  <label htmlFor="email">Email</label>
  <input id="email" autoComplete="email" />
</div>
```

**3. Embed JavaScript expressions**

Curly braces evaluate any JavaScript expression — variables, function calls, ternaries, template literals:

```tsx
function ProductPrice({ price, currency }: { price: number; currency: string }) {
  const formatted = new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency,
  }).format(price);

  return <span className="price">{formatted}</span>;
}
```

**4. Inline styles use objects**

The `style` attribute takes a JavaScript object with camelCased properties, not a CSS string:

```tsx
<div style={{ backgroundColor: '#f0f0f0', padding: '1rem' }}>
  Styled content
</div>
```

## Common Pitfalls

1. **Using `class` instead of `className`** — `class` is a reserved word in JavaScript. JSX uses `className` for CSS classes. Your IDE or linter will catch this, but it trips up beginners frequently.
2. **Forgetting to close self-closing tags** — Unlike HTML, JSX requires `<img />` and `<input />` to include the closing slash. Omitting it causes a syntax error.

## Best Practices

1. **Prefer fragments over wrapper divs** — Extra `<div>` elements pollute the DOM and can break CSS layouts. Use `<></>` when you need to return siblings without a semantic container.
2. **Extract complex expressions into variables** — If a JSX expression is getting long, assign it to a `const` above the return statement for readability.

## Summary

- JSX is a syntax extension that compiles to JavaScript function calls producing React elements.
- Use curly braces `{}` to embed any JavaScript expression inside markup.
- Attributes use camelCase (`className`, `onClick`, `htmlFor`) and all tags must be explicitly closed.

## Code Examples

**JSX with dynamic expressions, className, and inline styles**

```tsx
function Profile() {
  const user = { name: 'Alice', imageUrl: '/avatar.png' };

  return (
    <div className="profile">
      <img
        src={user.imageUrl}
        alt={user.name}
        style={{ borderRadius: '50%' }}
      />
      <h2>{user.name}</h2>
    </div>
  );
}
```


## Resources

- [Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx) — Official guide to JSX syntax and rules

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*