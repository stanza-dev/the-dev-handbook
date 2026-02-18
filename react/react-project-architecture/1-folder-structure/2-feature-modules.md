---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-feature-modules"
---

# Feature Module Pattern

Organize code by business domain.

## Feature Structure

```
features/
├── auth/
│   ├── components/
│   │   ├── LoginForm.tsx
│   │   ├── RegisterForm.tsx
│   │   └── UserMenu.tsx
│   ├── hooks/
│   │   ├── useAuth.ts
│   │   └── usePermissions.ts
│   ├── services/
│   │   └── authApi.ts
│   ├── store/
│   │   └── authStore.ts
│   ├── types.ts
│   └── index.ts        # Public API
│
├── products/
│   ├── components/
│   │   ├── ProductCard.tsx
│   │   ├── ProductList.tsx
│   │   └── ProductFilters.tsx
│   ├── hooks/
│   │   └── useProducts.ts
│   ├── services/
│   │   └── productsApi.ts
│   └── index.ts
│
└── cart/
    ├── components/
    ├── hooks/
    └── index.ts
```

## Public API (index.ts)

```tsx
// features/auth/index.ts

// Components
export { LoginForm } from './components/LoginForm';
export { UserMenu } from './components/UserMenu';

// Hooks
export { useAuth } from './hooks/useAuth';
export { usePermissions } from './hooks/usePermissions';

// Types
export type { User, AuthState } from './types';

// Don't export internal implementation
// ❌ export { authApi } from './services/authApi';
```

## Using Features

```tsx
// In app or other features
import { LoginForm, useAuth, User } from '@/features/auth';
import { ProductList, useProducts } from '@/features/products';
import { CartIcon, useCart } from '@/features/cart';

function App() {
  const { user, isAuthenticated } = useAuth();
  
  return (
    <div>
      {isAuthenticated ? (
        <>
          <UserMenu />
          <ProductList />
          <CartIcon />
        </>
      ) : (
        <LoginForm />
      )}
    </div>
  );
}
```

## Feature Dependencies

```
                ┌─────────┐
                │  shared │
                └────┬────┘
                     │
     ┌───────────────┼───────────────┐
     │               │               │
┌────▼────┐    ┌────▼────┐    ┌────▼────┐
│  auth   │    │products │    │  cart   │
└────┬────┘    └─────────┘    └────┬────┘
     │                              │
     │         ┌──────────┐         │
     └────────►│ checkout │◄────────┘
               └──────────┘
```

**Rules**:
- Features can import from `shared`
- Features can import from other features' public API
- Avoid circular dependencies
- Features should be deletable without breaking the app

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*