---
source_course: "react-project-architecture"
source_lesson: "react-project-architecture-circular-dependencies"
---

# Avoiding Circular Dependencies

Circular dependencies cause subtle bugs and maintenance issues.

## Detecting Circular Dependencies

```bash
# Using madge
npx madge --circular src/

# With visualization
npx madge --circular --image graph.svg src/
```

## Common Patterns That Cause Cycles

### Pattern 1: Shared Types

```tsx
// ❌ Circular: A imports B, B imports A

// features/auth/types.ts
import { Product } from '@/features/products/types';
export type User = { favoriteProducts: Product[] };

// features/products/types.ts
import { User } from '@/features/auth/types';
export type Product = { createdBy: User };
```

**Solution**: Shared types module

```tsx
// shared/types/index.ts
export type UserId = string;
export type ProductId = string;

// features/auth/types.ts
import { ProductId } from '@/shared/types';
export type User = { favoriteProductIds: ProductId[] };

// features/products/types.ts
import { UserId } from '@/shared/types';
export type Product = { createdById: UserId };
```

### Pattern 2: Utility Functions

```tsx
// ❌ Circular
// utils/format.ts
import { config } from './config';

// utils/config.ts
import { formatDate } from './format';
```

**Solution**: Dependency injection or extract shared code

```tsx
// utils/format.ts
export const formatDate = (date, locale) => { /* ... */ };

// utils/config.ts
import { formatDate } from './format';
const locale = 'en-US';
export const formatConfigDate = (date) => formatDate(date, locale);
```

### Pattern 3: Cross-Feature Dependencies

```tsx
// ❌ Circular features
// features/orders/hooks/useOrder.ts
import { useCart } from '@/features/cart';

// features/cart/hooks/useCart.ts
import { useOrders } from '@/features/orders';
```

**Solution**: Event-based communication

```tsx
// shared/events/index.ts
export const events = {
  ORDER_CREATED: 'order:created',
  CART_CLEARED: 'cart:cleared',
};

// features/orders/hooks/useOrder.ts
import { useEventEmitter } from '@/shared/hooks';

function useOrder() {
  const emit = useEventEmitter();
  
  const createOrder = async () => {
    await api.createOrder();
    emit('order:created');
  };
}

// features/cart/hooks/useCart.ts
import { useEventListener } from '@/shared/hooks';

function useCart() {
  useEventListener('order:created', () => {
    clearCart();
  });
}
```

## Prevention Strategies

1. **Type-only imports**: `import type { User } from ...`
2. **Shared types module**: Common types in shared/
3. **Interface segregation**: Define interfaces at boundaries
4. **Events over direct imports**: Decouple features

---

> 📘 *This lesson is part of the [React Project Architecture](https://stanza.dev/courses/react-project-architecture) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*