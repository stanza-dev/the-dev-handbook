---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-choosing-state-library"
---

# Choosing a State Library

Decide based on your needs.

## Context API

**Best for:**
- Theme/locale
- Auth state
- Small apps
- Infrequent updates

```jsx
// Simple, built-in, no dependencies
const ThemeContext = createContext();
```

## Zustand

**Best for:**
- Medium-large apps
- Simple mental model
- Need devtools/persistence
- Team familiarity with Redux patterns

```jsx
// Single store, familiar API
const useStore = create((set) => ({
  count: 0,
  increment: () => set((s) => ({ count: s.count + 1 })),
}));
```

## Jotai

**Best for:**
- Fine-grained reactivity
- Lots of derived state
- Suspense integration
- Bottom-up state design

```jsx
// Atoms compose naturally
const countAtom = atom(0);
const doubleAtom = atom((get) => get(countAtom) * 2);
```

## Redux Toolkit

**Best for:**
- Very large apps
- Complex business logic
- Strong typing needs
- Time-travel debugging
- Team with Redux experience

```jsx
// Powerful but more boilerplate
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => { state.value += 1; },
  },
});
```

## Recoil

**Best for:**
- Facebook-style apps
- Graph-based state
- Concurrent mode
- Experimental features

## Decision Matrix

```
Q: Is it server data?
├─ Yes → TanStack Query / SWR
└─ No → Continue

Q: How many components share this state?
├─ 1-2 → useState + props
├─ Small subtree → Context
└─ Many across app → External store

Q: How often does it change?
├─ Rarely → Context is fine
└─ Frequently → External store

Q: Do you need persistence/devtools?
├─ Yes → Zustand/Redux
└─ No → Context or Jotai

Q: Do you have lots of derived state?
├─ Yes → Jotai
└─ No → Zustand
```

## Mixing Solutions

You don't have to choose just one:

```jsx
function App() {
  return (
    // Theme rarely changes - Context
    <ThemeProvider>
      {/* User session - Zustand for persistence */}
      {/* Cart - Zustand or Context + useReducer */}
      {/* Server data - TanStack Query */}
      <QueryClientProvider client={queryClient}>
        <Router />
      </QueryClientProvider>
    </ThemeProvider>
  );
}
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*