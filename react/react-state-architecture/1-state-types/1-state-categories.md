---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-state-categories"
---

# Categories of State

Not all state is created equal. Understanding state types helps choose the right solution.

## UI State

State that affects visual presentation:

```jsx
// Modal open/closed
const [isOpen, setIsOpen] = useState(false);

// Tab selection
const [activeTab, setActiveTab] = useState('profile');

// Form input values
const [name, setName] = useState('');

// Hover/focus states
const [isHovered, setIsHovered] = useState(false);
```

**Characteristics**:
- Component-local
- Often ephemeral
- Rarely shared
- Use `useState` or `useReducer`

## Server State

Data fetched from backend APIs:

```jsx
// User profile from API
const { data: user, isLoading } = useQuery(['user', id], fetchUser);

// List of posts
const { data: posts } = useQuery('posts', fetchPosts);
```

**Characteristics**:
- Owned by backend
- Cached on client
- Needs synchronization
- Has loading/error states
- Use TanStack Query, SWR

## Client State

Application state not from server:

```jsx
// Shopping cart
const [cartItems, setCartItems] = useState([]);

// User preferences
const [theme, setTheme] = useState('light');

// Authentication status
const [isAuthenticated, setIsAuthenticated] = useState(false);
```

**Characteristics**:
- Owned by client
- May persist locally
- Shared across components
- Use Context, Zustand, Redux

## Form State

Input values and validation:

```jsx
// Controlled form
const [formData, setFormData] = useState({
  email: '',
  password: '',
});
const [errors, setErrors] = useState({});
const [isSubmitting, setIsSubmitting] = useState(false);
```

**Characteristics**:
- Complex validation logic
- Touch/dirty tracking
- Often uses libraries
- Use React Hook Form, Formik

## URL State

State encoded in the URL:

```jsx
// From URL params
const { productId } = useParams();

// From query string
const searchParams = useSearchParams();
const page = searchParams.get('page');
const filter = searchParams.get('filter');
```

**Characteristics**:
- Shareable/bookmarkable
- Browser history integration
- Use router params/search params

## Decision Framework

| State Type | Scope | Solution |
|------------|-------|----------|
| UI State | Single component | `useState` |
| UI State | Component tree | `useReducer` + Context |
| Server State | App-wide | TanStack Query/SWR |
| Client State | App-wide | Context/Zustand/Redux |
| Form State | Form component | React Hook Form |
| URL State | App-wide | Router |

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*