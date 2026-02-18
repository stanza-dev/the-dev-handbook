---
source_course: "react-state-architecture"
source_lesson: "react-state-architecture-compound-context"
---

# Context with useReducer

Combine Context with useReducer for complex state.

## The Pattern

```jsx
import { createContext, useContext, useReducer, useMemo } from 'react';

// Action types
const ACTIONS = {
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
  CLEAR_CART: 'CLEAR_CART',
};

// Reducer
function cartReducer(state, action) {
  switch (action.type) {
    case ACTIONS.ADD_ITEM:
      const existing = state.items.find(i => i.id === action.payload.id);
      if (existing) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          ),
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }],
      };
    case ACTIONS.REMOVE_ITEM:
      return {
        ...state,
        items: state.items.filter(i => i.id !== action.payload),
      };
    case ACTIONS.CLEAR_CART:
      return { ...state, items: [] };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

// Initial state
const initialState = {
  items: [],
};

// Contexts
const CartStateContext = createContext(null);
const CartDispatchContext = createContext(null);

// Provider
function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  return (
    <CartStateContext.Provider value={state}>
      <CartDispatchContext.Provider value={dispatch}>
        {children}
      </CartDispatchContext.Provider>
    </CartStateContext.Provider>
  );
}

// Hooks
function useCartState() {
  const context = useContext(CartStateContext);
  if (context === null) {
    throw new Error('useCartState must be used within CartProvider');
  }
  return context;
}

function useCartDispatch() {
  const context = useContext(CartDispatchContext);
  if (context === null) {
    throw new Error('useCartDispatch must be used within CartProvider');
  }
  return context;
}

// Convenience hook with actions
function useCart() {
  const state = useCartState();
  const dispatch = useCartDispatch();
  
  const actions = useMemo(() => ({
    addItem: (item) => dispatch({ type: ACTIONS.ADD_ITEM, payload: item }),
    removeItem: (id) => dispatch({ type: ACTIONS.REMOVE_ITEM, payload: id }),
    clearCart: () => dispatch({ type: ACTIONS.CLEAR_CART }),
  }), [dispatch]);
  
  const total = useMemo(() =>
    state.items.reduce((sum, item) => sum + item.price * item.quantity, 0),
    [state.items]
  );
  
  return {
    items: state.items,
    total,
    ...actions,
  };
}

export { CartProvider, useCart, useCartState, useCartDispatch };
```

## Usage

```jsx
// App.jsx
import { CartProvider } from './cart-context';

function App() {
  return (
    <CartProvider>
      <Header />
      <ProductList />
    </CartProvider>
  );
}

// CartIcon.jsx - Only reads state
function CartIcon() {
  const { items } = useCartState();
  return <span>Cart ({items.length})</span>;
}

// AddToCartButton.jsx - Only dispatches
function AddToCartButton({ product }) {
  const { addItem } = useCart();
  return (
    <button onClick={() => addItem(product)}>
      Add to Cart
    </button>
  );
}

// CartSummary.jsx - Reads computed value
function CartSummary() {
  const { items, total, clearCart } = useCart();
  
  return (
    <div>
      <ul>
        {items.map(item => (
          <li key={item.id}>
            {item.name} × {item.quantity}
          </li>
        ))}
      </ul>
      <p>Total: ${total.toFixed(2)}</p>
      <button onClick={clearCart}>Clear Cart</button>
    </div>
  );
}
```

---

> 📘 *This lesson is part of the [React State Architecture](https://stanza.dev/courses/react-state-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*