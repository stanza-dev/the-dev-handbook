---
source_course: "react-typescript"
source_lesson: "react-typescript-custom-event-props"
---

# Custom Event Props

Typing callback props that your components accept.

## Simple Callback Props

```tsx
type ButtonProps = {
  onClick: () => void;  // No arguments, no return
};

function Button({ onClick }: ButtonProps) {
  return <button onClick={onClick}>Click</button>;
}
```

## Callback with Arguments

```tsx
type UserCardProps = {
  user: User;
  onSelect: (userId: string) => void;
  onEdit: (user: User) => void;
  onDelete: (userId: string) => Promise<void>;
};

function UserCard({ user, onSelect, onEdit, onDelete }: UserCardProps) {
  return (
    <div onClick={() => onSelect(user.id)}>
      <h3>{user.name}</h3>
      <button onClick={() => onEdit(user)}>Edit</button>
      <button onClick={() => onDelete(user.id)}>Delete</button>
    </div>
  );
}
```

## Optional Callbacks

```tsx
type ModalProps = {
  isOpen: boolean;
  onClose: () => void;       // Required
  onConfirm?: () => void;    // Optional
  onCancel?: () => void;     // Optional
};

function Modal({ isOpen, onClose, onConfirm, onCancel }: ModalProps) {
  return (
    <div>
      {/* Always available */}
      <button onClick={onClose}>×</button>
      
      {/* Check before calling */}
      {onConfirm && <button onClick={onConfirm}>Confirm</button>}
      {onCancel && <button onClick={onCancel}>Cancel</button>}
    </div>
  );
}
```

## Passing Event Through

```tsx
type SearchInputProps = {
  value: string;
  onChange: (value: string) => void;
  onSubmit: () => void;
};

function SearchInput({ value, onChange, onSubmit }: SearchInputProps) {
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    onChange(e.target.value);
  };

  const handleKeyDown = (e: React.KeyboardEvent) => {
    if (e.key === 'Enter') {
      onSubmit();
    }
  };

  return (
    <input
      value={value}
      onChange={handleChange}
      onKeyDown={handleKeyDown}
    />
  );
}

// Parent component - clean API
<SearchInput
  value={query}
  onChange={setQuery}  // Just receives the string
  onSubmit={handleSearch}
/>
```

## Generic Event Props

```tsx
type SelectProps<T> = {
  options: T[];
  value: T;
  onChange: (value: T) => void;
  getLabel: (item: T) => string;
  getValue: (item: T) => string;
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
      value={getValue(value)}
      onChange={e => {
        const selected = options.find(o => getValue(o) === e.target.value);
        if (selected) onChange(selected);
      }}
    >
      {options.map(option => (
        <option key={getValue(option)} value={getValue(option)}>
          {getLabel(option)}
        </option>
      ))}
    </select>
  );
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*