---
source_course: "react"
source_lesson: "react-component-patterns"
---

# Component Patterns

## Introduction

As your React application grows, you will encounter recurring design challenges: how to share layout logic, how to enhance a component's behavior without modifying it, and how to make components flexible enough for different contexts. This lesson covers the most important component patterns used in production React applications.

## Key Concepts

- **Controlled vs. Uncontrolled Components**: Components whose values are driven by React state (controlled) versus components that manage their own internal state (uncontrolled).
- **Compound Components**: A set of components that work together to form a cohesive unit (like `<Select>` with `<Option>` children).
- **Render Props**: A pattern where a component receives a function as a prop and calls it to determine what to render.
- **Layout Components**: Components focused on arrangement and spacing rather than content.

## Real World Context

A form library like React Hook Form uses the controlled/uncontrolled distinction at its core. A tabs component in a UI library uses compound components so that `<Tabs>`, `<Tab>`, and `<TabPanel>` communicate implicitly. Understanding these patterns lets you design APIs that are intuitive to consume and powerful enough for complex use cases.

## Deep Dive

### Controlled Components

A controlled component derives its displayed value entirely from React state:

```tsx
function SearchInput() {
  const [query, setQuery] = useState('');

  const handleChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    setQuery(event.target.value);
  };

  return (
    <input
      type="text"
      value={query}
      onChange={handleChange}
      placeholder="Search..."
    />
  );
}
```

Because React controls the value, you can validate, transform, or restrict input in real time.

### Uncontrolled Components with Refs

An uncontrolled component lets the DOM handle the value. You read it only when needed:

```tsx
function FileUpload() {
  const fileInputRef = useRef<HTMLInputElement>(null);

  const handleSubmit = (event: React.FormEvent) => {
    event.preventDefault();
    const file = fileInputRef.current?.files?.[0];
    if (file) {
      uploadFile(file);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="file" ref={fileInputRef} />
      <button type="submit">Upload</button>
    </form>
  );
}
```

### Compound Components

Compound components share implicit state through Context:

```tsx
const AccordionContext = createContext<{
  openIndex: number | null;
  toggle: (index: number) => void;
} | null>(null);

function Accordion({ children }: { children: React.ReactNode }) {
  const [openIndex, setOpenIndex] = useState<number | null>(null);
  const toggle = (index: number) => {
    setOpenIndex(prev => (prev === index ? null : index));
  };

  return (
    <AccordionContext value={{ openIndex, toggle }}>
      <div className="accordion">{children}</div>
    </AccordionContext>
  );
}

function AccordionItem({ index, title, children }: {
  index: number;
  title: string;
  children: React.ReactNode;
}) {
  const ctx = useContext(AccordionContext);
  if (!ctx) throw new Error('AccordionItem must be inside Accordion');
  const isOpen = ctx.openIndex === index;

  return (
    <div>
      <button onClick={() => ctx.toggle(index)}>{title}</button>
      {isOpen && <div className="accordion-body">{children}</div>}
    </div>
  );
}
```

### Layout Components

Components focused on spatial arrangement keep layout concerns separate from business logic:

```tsx
function Stack({ gap = '1rem', children }: { gap?: string; children: React.ReactNode }) {
  return (
    <div style={{ display: 'flex', flexDirection: 'column', gap }}>
      {children}
    </div>
  );
}
```

## Common Pitfalls

1. **Mixing controlled and uncontrolled patterns** — Setting both `value` and `defaultValue` on an input, or switching between them, causes confusing bugs. Pick one approach and stick with it.
2. **Overusing render props when composition suffices** — Render props add complexity. Use children or composition first; reach for render props only when the child needs data from the parent's internal logic.

## Best Practices

1. **Default to controlled components for forms** — Controlled components give you full control over validation, formatting, and submission logic.
2. **Use compound components for tightly coupled groups** — When several components must share state (Tabs + Tab + TabPanel), the compound pattern keeps the API clean.

## Summary

- Controlled components derive their value from state; uncontrolled components let the DOM manage the value.
- Compound components share implicit state through Context, creating cohesive multi-part UI elements.
- Layout components separate spatial concerns from business logic, improving reusability.

## Code Examples

**A controlled search input component with form submission**

```tsx
function SearchInput({ onSearch }: { onSearch: (query: string) => void }) {
  const [query, setQuery] = useState('');

  const handleSubmit = (event: React.FormEvent) => {
    event.preventDefault();
    onSearch(query);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        type="text"
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="Search products..."
      />
      <button type="submit">Search</button>
    </form>
  );
}
```


## Resources

- [Sharing State Between Components](https://react.dev/learn/sharing-state-between-components) — Official guide on lifting state up and sharing between components

---

> 📘 *This lesson is part of the [React Basics](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*