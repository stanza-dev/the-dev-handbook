---
source_course: "react-typescript"
source_lesson: "react-typescript-generic-context"
---

# Generic Context Pattern

Creating reusable, type-safe context factories.

## Generic Context Factory

```tsx
import { createContext, useContext, ReactNode } from 'react';

function createSafeContext<T>(displayName: string) {
  const Context = createContext<T | null>(null);
  Context.displayName = displayName;

  function useContextSafe() {
    const context = useContext(Context);
    if (!context) {
      throw new Error(
        `use${displayName} must be used within ${displayName}Provider`
      );
    }
    return context;
  }

  return [Context.Provider, useContextSafe] as const;
}

// Usage
type ThemeContextType = {
  theme: 'light' | 'dark';
  toggleTheme: () => void;
};

const [ThemeProvider, useTheme] = createSafeContext<ThemeContextType>('Theme');

export { ThemeProvider, useTheme };
```

## Context with Reducer

```tsx
type State = {
  items: string[];
  loading: boolean;
};

type Action =
  | { type: 'ADD_ITEM'; payload: string }
  | { type: 'REMOVE_ITEM'; payload: string }
  | { type: 'SET_LOADING'; payload: boolean };

type ContextType = {
  state: State;
  dispatch: React.Dispatch<Action>;
};

const ItemsContext = createContext<ContextType | null>(null);

function itemsReducer(state: State, action: Action): State {
  switch (action.type) {
    case 'ADD_ITEM':
      return { ...state, items: [...state.items, action.payload] };
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(i => i !== action.payload)
      };
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
  }
}

function ItemsProvider({ children }: { children: ReactNode }) {
  const [state, dispatch] = useReducer(itemsReducer, {
    items: [],
    loading: false
  });

  return (
    <ItemsContext.Provider value={{ state, dispatch }}>
      {children}
    </ItemsContext.Provider>
  );
}

function useItems() {
  const context = useContext(ItemsContext);
  if (!context) {
    throw new Error('useItems must be used within ItemsProvider');
  }
  return context;
}
```

## Combining Multiple Contexts

```tsx
type AllProviderProps = {
  children: ReactNode;
};

function AllProviders({ children }: AllProviderProps) {
  return (
    <AuthProvider>
      <ThemeProvider>
        <ItemsProvider>
          {children}
        </ItemsProvider>
      </ThemeProvider>
    </AuthProvider>
  );
}
```

---

> 📘 *This lesson is part of the [React TypeScript Mastery](https://stanza.dev/courses/react-typescript) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*