---
source_course: "react"
source_lesson: "react-jsx-basics"
---

# JSX Basics

JSX is a syntax extension for JavaScript that looks like HTML. It produces React "elements".

## Key Rules

1. **Return a single root element**: Wrap multiple elements in a parent or use fragments `<></>`
2. **Close all tags**: Self-closing tags need `/>`
3. **camelCase for attributes**: `className` instead of `class`, `onClick` instead of `onclick`

## Embedding Expressions

Use curly braces `{}` to embed any JavaScript expression in JSX.

## Code Examples

**JSX with expressions and inline styles**

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

- [Writing Markup with JSX](https://react.dev/learn/writing-markup-with-jsx) — Official guide to JSX

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*