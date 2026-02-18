---
source_course: "react-typescript"
source_lesson: "react-typescript-typing-usestate"
---

# Typing useState

TypeScript often infers useState types, but sometimes you need to be explicit.

## Automatic Inference

```tsx
// Type inferred as string
const [name, setName] = useState('Alice');

// Type inferred as number
const [count, setCount] = useState(0);

// Type inferred as boolean
const [isOpen, setIsOpen] = useState(false);
```

## Explicit Type Annotation

Needed when initial value doesn't represent all possible types:

```tsx
// Initial null, but will be User later
const [user, setUser] = useState<User | null>(null);

// Empty array that will hold Users
const [users, setUsers] = useState<User[]>([]);

// Union type
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');
```

## Complex Object State

```tsx
type FormState = {
  name: string;
  email: string;
  age: number | null;
};

const [form, setForm] = useState<FormState>({
  name: '',
  email: '',
  age: null
});

// Type-safe updates
setForm(prev => ({ ...prev, name: 'Alice' }));

// Error: 'nam' doesn't exist
setForm(prev => ({ ...prev, nam: 'Alice' }));
```

## Avoiding Type Assertions

```tsx
// ❌ Bad: Type assertion hides errors
const [user, setUser] = useState({} as User);

// ✅ Good: Explicit null state
const [user, setUser] = useState<User | null>(null);

// Usage requires null check
if (user) {
  console.log(user.name); // Safe
}
```

## Lazy Initialization

```tsx
// Expensive computation typed correctly
const [data, setData] = useState<ComplexData>(() => {
  return computeInitialData();
});
```

## Common Patterns

```tsx
// Toggle state
const [isVisible, setIsVisible] = useState(false);
const toggle = () => setIsVisible(prev => !prev);

// Counter
const [count, setCount] = useState(0);
const increment = () => setCount(prev => prev + 1);

// Input state
const [value, setValue] = useState('');
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*