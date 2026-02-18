---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-feature-based-state"
---

# Feature-Based State Organization

Organize state by feature, not by type.

## Directory Structure

```
src/
├── features/
│   ├── auth/
│   │   ├── components/
│   │   │   ├── LoginForm.tsx
│   │   │   └── UserMenu.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── store/
│   │   │   ├── authStore.ts
│   │   │   └── authSlice.ts
│   │   └── index.ts
│   │
│   ├── cart/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── store/
│   │   └── index.ts
│   │
│   └── products/
│       ├── components/
│       ├── hooks/
│       ├── store/
│       └── index.ts
│
├── shared/
│   ├── components/
│   ├── hooks/
│   └── utils/
│
└── app/
    ├── store.ts
    └── App.tsx
```

## Feature Module Pattern

```tsx
// features/cart/store/cartStore.ts
import { create } from 'zustand';
import { persist } from 'zustand/middleware';

type CartItem = {
  id: string;
  name: string;
  price: number;
  quantity: number;
};

type CartStore = {
  items: CartItem[];
  addItem: (item: Omit<CartItem, 'quantity'>) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
};

export const useCartStore = create<CartStore>()(
  persist(
    (set, get) => ({
      items: [],
      addItem: (item) =>
        set((state) => {
          const existing = state.items.find((i) => i.id === item.id);
          if (existing) {
            return {
              items: state.items.map((i) =>
                i.id === item.id ? { ...i, quantity: i.quantity + 1 } : i
              ),
            };
          }
          return { items: [...state.items, { ...item, quantity: 1 }] };
        }),
      removeItem: (id) =>
        set((state) => ({
          items: state.items.filter((i) => i.id !== id),
        })),
      updateQuantity: (id, quantity) =>
        set((state) => ({
          items: state.items.map((i) =>
            i.id === id ? { ...i, quantity } : i
          ),
        })),
      clearCart: () => set({ items: [] }),
    }),
    { name: 'cart-storage' }
  )
);

// Selectors
export const selectCartTotal = (state: CartStore) =>
  state.items.reduce((sum, item) => sum + item.price * item.quantity, 0);

export const selectCartCount = (state: CartStore) =>
  state.items.reduce((sum, item) => sum + item.quantity, 0);
```

```tsx
// features/cart/hooks/useCart.ts
import { useCartStore, selectCartTotal, selectCartCount } from '../store/cartStore';

export function useCart() {
  const items = useCartStore((state) => state.items);
  const total = useCartStore(selectCartTotal);
  const count = useCartStore(selectCartCount);
  const addItem = useCartStore((state) => state.addItem);
  const removeItem = useCartStore((state) => state.removeItem);
  const clearCart = useCartStore((state) => state.clearCart);

  return {
    items,
    total,
    count,
    addItem,
    removeItem,
    clearCart,
  };
}
```

```tsx
// features/cart/index.ts - Public API
export { useCart } from './hooks/useCart';
export { CartIcon } from './components/CartIcon';
export { CartDrawer } from './components/CartDrawer';
// Don't export internal implementation details
```

## Using Features

```tsx
// app/pages/ProductPage.tsx
import { useCart } from '@/features/cart';
import { useProduct } from '@/features/products';

function ProductPage({ productId }: { productId: string }) {
  const { product, isLoading } = useProduct(productId);
  const { addItem } = useCart();

  if (isLoading) return <Loading />;

  return (
    <div>
      <h1>{product.name}</h1>
      <button onClick={() => addItem(product)}>
        Add to Cart
      </button>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*