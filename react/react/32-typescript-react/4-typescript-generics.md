---
source_course: "react"
source_lesson: "react-typescript-generics"
---

# Generic Components

## Introduction

Generic components accept type parameters, making them flexible and reusable while maintaining type safety. Think `Array<T>` but for React components.

## Key Concepts

**Generic components** use type parameters to create flexible, type-safe abstractions. The type flows through props, state, and callbacks.

## Real World Context

Generics are essential for:
- List/table components that work with any data type
- Form libraries that handle any shape
- Select/dropdown components with typed options
- Data fetching hooks and components

## Deep Dive

### Basic Generic Component

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T, index: number) => ReactNode;
  keyExtractor: (item: T) => string;
};

function List<T>({ items, renderItem, keyExtractor }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// Usage - T is inferred as User
<List
  items={users}
  renderItem={(user) => <span>{user.name}</span>}
  keyExtractor={(user) => user.id}
/>
```

### Generic Select Component

```tsx
type SelectProps<T> = {
  options: T[];
  value: T | null;
  onChange: (value: T) => void;
  getLabel: (option: T) => string;
  getValue: (option: T) => string;
};

function Select<T>({
  options,
  value,
  onChange,
  getLabel,
  getValue
}: SelectProps<T>) {
  return (
    <select
      value={value ? getValue(value) : ''}
      onChange={(e) => {
        const selected = options.find(
          opt => getValue(opt) === e.target.value
        );
        if (selected) onChange(selected);
      }}
    >
      <option value="">Select...</option>
      {options.map(option => (
        <option key={getValue(option)} value={getValue(option)}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
}
```

### Constraining Generics

```tsx
// T must have an 'id' property
type WithId = { id: string };

type DataTableProps<T extends WithId> = {
  data: T[];
  columns: Array<{
    key: keyof T;
    header: string;
  }>;
};

function DataTable<T extends WithId>({ data, columns }: DataTableProps<T>) {
  return (
    <table>
      <thead>
        <tr>
          {columns.map(col => (
            <th key={String(col.key)}>{col.header}</th>
          ))}
        </tr>
      </thead>
      <tbody>
        {data.map(row => (
          <tr key={row.id}>
            {columns.map(col => (
              <td key={String(col.key)}>
                {String(row[col.key])}
              </td>
            ))}
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

### Generic Hooks

```tsx
function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T) => void] {
  const [storedValue, setStoredValue] = useState<T>(() => {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : initialValue;
  });

  const setValue = (value: T) => {
    setStoredValue(value);
    localStorage.setItem(key, JSON.stringify(value));
  };

  return [storedValue, setValue];
}

// Usage with inference
const [settings, setSettings] = useLocalStorage('settings', { theme: 'dark' });
// settings is typed as { theme: string }
```

## Common Pitfalls

1. **Over-constraining**: Adding unnecessary constraints limits reusability.
2. **Not inferring when possible**: Let TypeScript infer from arguments rather than requiring explicit type parameters.
3. **Losing type information**: Using `any` in generics defeats the purpose.

## Best Practices

- Let TypeScript infer generic types from props when possible
- Use constraints (`extends`) only when you need to access specific properties
- Provide meaningful type parameter names (`T`, `TItem`, `TData`)
- Export the props type for consumers who need it
- Consider whether a generic is needed - sometimes a union type suffices

## Summary

Generic components create flexible, reusable abstractions while maintaining full type safety. Use them for lists, tables, selects, and any component that works with arbitrary data types. Let TypeScript infer when possible.

## Code Examples

**Generic components for async data and modals**

```tsx
import { useState, ReactNode } from 'react';

// Generic async data component
type AsyncDataProps<T> = {
  promise: Promise<T>;
  loading: ReactNode;
  error: (err: Error) => ReactNode;
  children: (data: T) => ReactNode;
};

function AsyncData<T>({
  promise,
  loading,
  error,
  children
}: AsyncDataProps<T>) {
  const [state, setState] = useState<
    | { status: 'loading' }
    | { status: 'success'; data: T }
    | { status: 'error'; error: Error }
  >({ status: 'loading' });

  // Effect to resolve promise...

  switch (state.status) {
    case 'loading':
      return <>{loading}</>;
    case 'error':
      return <>{error(state.error)}</>;
    case 'success':
      return <>{children(state.data)}</>;
  }
}

// Usage with full type inference
type Product = { id: string; name: string; price: number };

<AsyncData<Product>
  promise={fetchProduct(id)}
  loading={<Spinner />}
  error={(err) => <Error message={err.message} />}
>
  {(product) => (
    <div>
      <h1>{product.name}</h1>
      <p>${product.price}</p>
    </div>
  )}
</AsyncData>

// Generic modal/dialog component
type ModalProps<T> = {
  isOpen: boolean;
  onClose: () => void;
  onConfirm: (data: T) => void;
  initialData: T;
  children: (data: T, setData: (data: T) => void) => ReactNode;
};

function Modal<T>({
  isOpen,
  onClose,
  onConfirm,
  initialData,
  children
}: ModalProps<T>) {
  const [data, setData] = useState(initialData);

  if (!isOpen) return null;

  return (
    <div className="modal-backdrop">
      <div className="modal">
        {children(data, setData)}
        <div className="modal-actions">
          <button onClick={onClose}>Cancel</button>
          <button onClick={() => onConfirm(data)}>Confirm</button>
        </div>
      </div>
    </div>
  );
}
```


## Resources

- [TypeScript Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html) — TypeScript generics handbook

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*