---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-zustand-basics"
---

# Zustand: Simple State Management

Zustand is a minimal state management library.

## Installation

```bash
npm install zustand
```

## Creating a Store

```jsx
import { create } from 'zustand';

const useStore = create((set, get) => ({
  // State
  count: 0,
  user: null,
  
  // Actions
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
  
  // Async actions
  fetchUser: async (id) => {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    set({ user });
  },
  
  // Using get() to access current state
  incrementBy: (amount) => set({ count: get().count + amount }),
}));
```

## Using the Store

```jsx
// Use entire store
function Counter() {
  const { count, increment, decrement } = useStore();
  return (
    <div>
      <span>{count}</span>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
    </div>
  );
}

// Select specific value (optimized)
function CountDisplay() {
  const count = useStore((state) => state.count);
  return <span>Count: {count}</span>;
}

// Select multiple values
function UserInfo() {
  const { user, fetchUser } = useStore((state) => ({
    user: state.user,
    fetchUser: state.fetchUser,
  }));
  
  useEffect(() => {
    fetchUser('123');
  }, [fetchUser]);
  
  return <div>{user?.name}</div>;
}
```

## Slices Pattern

```jsx
// userSlice.js
export const createUserSlice = (set) => ({
  user: null,
  isAuthenticated: false,
  login: async (credentials) => {
    const user = await authApi.login(credentials);
    set({ user, isAuthenticated: true });
  },
  logout: () => set({ user: null, isAuthenticated: false }),
});

// cartSlice.js
export const createCartSlice = (set) => ({
  items: [],
  addItem: (item) => set((state) => ({
    items: [...state.items, item],
  })),
  clearCart: () => set({ items: [] }),
});

// store.js
import { create } from 'zustand';
import { createUserSlice } from './userSlice';
import { createCartSlice } from './cartSlice';

const useStore = create((...args) => ({
  ...createUserSlice(...args),
  ...createCartSlice(...args),
}));
```

## Middleware

```jsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

const useStore = create(
  devtools(
    persist(
      (set) => ({
        count: 0,
        increment: () => set((state) => ({ count: state.count + 1 })),
      }),
      {
        name: 'my-store', // localStorage key
      }
    )
  )
);
```

## TypeScript Support

```tsx
type State = {
  count: number;
  user: User | null;
};

type Actions = {
  increment: () => void;
  setUser: (user: User) => void;
};

const useStore = create<State & Actions>((set) => ({
  count: 0,
  user: null,
  increment: () => set((state) => ({ count: state.count + 1 })),
  setUser: (user) => set({ user }),
}));
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*