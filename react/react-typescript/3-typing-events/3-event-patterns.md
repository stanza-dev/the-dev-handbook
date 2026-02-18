---
source_course: "react-typescript"
source_lesson: "react-typescript-form-event-patterns"
---

# Form Event Patterns

Type-safe form handling patterns.

## Basic Form Submit

```tsx
function LoginForm() {
  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    
    // Access form data
    const formData = new FormData(e.currentTarget);
    const email = formData.get('email') as string;
    const password = formData.get('password') as string;
    
    login(email, password);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" />
      <input name="password" type="password" />
      <button type="submit">Login</button>
    </form>
  );
}
```

## Controlled Form with Types

```tsx
type FormData = {
  email: string;
  password: string;
  rememberMe: boolean;
};

function LoginForm({ onSubmit }: { onSubmit: (data: FormData) => void }) {
  const [formData, setFormData] = useState<FormData>({
    email: '',
    password: '',
    rememberMe: false
  });

  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement>
  ) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    onSubmit(formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
      />
      <input
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange}
      />
      <input
        name="rememberMe"
        type="checkbox"
        checked={formData.rememberMe}
        onChange={handleChange}
      />
      <button type="submit">Login</button>
    </form>
  );
}
```

## Multiple Input Types

```tsx
type InputChangeEvent = React.ChangeEvent<
  HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement
>;

function handleChange(e: InputChangeEvent) {
  const { name, value } = e.target;
  setForm(prev => ({ ...prev, [name]: value }));
}
```

## Type-Safe Form Fields

```tsx
type FieldProps<T> = {
  name: keyof T;
  value: T[keyof T];
  onChange: (name: keyof T, value: string) => void;
};

function TextField<T>({ name, value, onChange }: FieldProps<T>) {
  return (
    <input
      name={name as string}
      value={value as string}
      onChange={e => onChange(name, e.target.value)}
    />
  );
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*