---
source_course: "react-hooks-deep-dive"
source_lesson: "react-hooks-deep-dive-useid"
---

# useId: Stable Unique IDs

`useId` generates unique IDs that are stable across server and client, avoiding hydration mismatches.

## The Problem It Solves

```tsx
// 🔴 Bad - IDs might mismatch between server and client
let nextId = 0;
function Field() {
  const id = `field-${nextId++}`; // Different on server vs client!
  return (
    <>
      <label htmlFor={id}>Name</label>
      <input id={id} />
    </>
  );
}
```

## The Solution

```tsx
import { useId } from 'react';

function Field({ label }: { label: string }) {
  const id = useId();
  
  return (
    <>
      <label htmlFor={id}>{label}</label>
      <input id={id} />
    </>
  );
}
```

## Multiple IDs from One Hook

```tsx
function PasswordField() {
  const id = useId();
  
  return (
    <>
      <label htmlFor={`${id}-password`}>Password</label>
      <input id={`${id}-password`} type="password" aria-describedby={`${id}-hint`} />
      <p id={`${id}-hint`}>Must be at least 12 characters</p>
    </>
  );
}
```

## Accessibility Patterns

```tsx
function Tooltip({ content, children }: TooltipProps) {
  const id = useId();
  const [isOpen, setIsOpen] = useState(false);

  return (
    <div
      onMouseEnter={() => setIsOpen(true)}
      onMouseLeave={() => setIsOpen(false)}
    >
      <span aria-describedby={isOpen ? id : undefined}>
        {children}
      </span>
      {isOpen && (
        <div id={id} role="tooltip">
          {content}
        </div>
      )}
    </div>
  );
}
```

## Important Notes

⚠️ **Do NOT use for list keys:**

```tsx
// 🔴 Wrong - keys should come from data
{items.map(() => {
  const id = useId(); // Called in a loop - breaks rules of hooks!
  return <li key={id}>...</li>;
})}

// ✅ Correct - use data IDs for keys
{items.map((item) => (
  <li key={item.id}>...</li>
))}
```

⚠️ **IDs include colons** (`:r1:`, `:r2:`) - this is intentional and valid for HTML IDs.

## Resources

- [useId API Reference](https://react.dev/reference/react/useId) — Official React documentation for useId hook

---

> 📘 *This lesson is part of the [React Hooks Deep Dive](https://stanza.dev/courses/react-hooks-deep-dive) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*