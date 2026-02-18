---
source_course: "react-typescript"
source_lesson: "react-typescript-discriminated-union-props"
---

# Discriminated Union Props

Props that change based on a discriminator value.

## Basic Pattern

```tsx
// Different props based on 'variant'
type AlertProps =
  | { variant: 'success'; message: string }
  | { variant: 'error'; message: string; onRetry: () => void }
  | { variant: 'warning'; message: string; onDismiss?: () => void };

function Alert(props: AlertProps) {
  switch (props.variant) {
    case 'success':
      return <div className="alert-success">{props.message}</div>;
    
    case 'error':
      return (
        <div className="alert-error">
          {props.message}
          <button onClick={props.onRetry}>Retry</button>
        </div>
      );
    
    case 'warning':
      return (
        <div className="alert-warning">
          {props.message}
          {props.onDismiss && (
            <button onClick={props.onDismiss}>Dismiss</button>
          )}
        </div>
      );
  }
}

// Usage
<Alert variant="success" message="Saved!" />
<Alert variant="error" message="Failed" onRetry={() => {}} />
<Alert variant="warning" message="Careful!" />

// TypeScript error: onRetry is required for error variant
<Alert variant="error" message="Failed" /> // Error!
```

## Mode-Based Props

```tsx
type InputProps = (
  | { mode: 'text'; value: string; onChange: (value: string) => void }
  | { mode: 'number'; value: number; onChange: (value: number) => void; min?: number; max?: number }
  | { mode: 'select'; value: string; onChange: (value: string) => void; options: string[] }
);

function Input(props: InputProps) {
  switch (props.mode) {
    case 'text':
      return (
        <input
          type="text"
          value={props.value}
          onChange={e => props.onChange(e.target.value)}
        />
      );
    
    case 'number':
      return (
        <input
          type="number"
          value={props.value}
          min={props.min}
          max={props.max}
          onChange={e => props.onChange(Number(e.target.value))}
        />
      );
    
    case 'select':
      return (
        <select
          value={props.value}
          onChange={e => props.onChange(e.target.value)}
        >
          {props.options.map(opt => (
            <option key={opt} value={opt}>{opt}</option>
          ))}
        </select>
      );
  }
}
```

## Mutually Exclusive Props

```tsx
// Either 'items' OR 'children', never both
type ListProps =
  | { items: string[]; children?: never }
  | { items?: never; children: ReactNode };

function List(props: ListProps) {
  if (props.items) {
    return (
      <ul>
        {props.items.map(item => <li key={item}>{item}</li>)}
      </ul>
    );
  }
  
  return <ul>{props.children}</ul>;
}

// Valid
<List items={['a', 'b', 'c']} />
<List><li>Custom item</li></List>

// Error: Can't have both
<List items={['a']}><li>b</li></List>
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*