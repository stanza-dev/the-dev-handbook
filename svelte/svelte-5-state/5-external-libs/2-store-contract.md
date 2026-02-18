---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-store-contract"
---

# The Universal Store Contract

Svelte's `$` prefix works with ANY object that has a `subscribe` method following the store contract. This means you can integrate external libraries!

## The Store Contract

```typescript
interface Store<T> {
  subscribe(callback: (value: T) => void): () => void;
}
```

- `subscribe` receives a callback
- Callback is called immediately with current value
- Callback is called again whenever value changes
- `subscribe` returns an unsubscribe function

## Creating Compatible Stores

Any object meeting this contract works with `$`:

```javascript
// A simple timer "store"
const timer = {
  subscribe(callback) {
    let seconds = 0;
    callback(seconds); // Call immediately
    
    const interval = setInterval(() => {
      seconds++;
      callback(seconds);
    }, 1000);
    
    return () => clearInterval(interval); // Cleanup
  }
};
```

```svelte
<script>
  import { timer } from './timer.js';
</script>

<p>Seconds: {$timer}</p> <!-- Works! -->
```

## Wrapping External Libraries

Wrap any observable/signal library:

```javascript
// Wrap RxJS Observable
import { Observable } from 'rxjs';

function fromObservable(observable) {
  return {
    subscribe(callback) {
      const subscription = observable.subscribe(callback);
      return () => subscription.unsubscribe();
    }
  };
}

// Usage
const myStore = fromObservable(someRxjsObservable$);
```

```javascript
// Wrap Zustand store
import { create } from 'zustand';

const useStore = create((set) => ({
  count: 0,
  increment: () => set(state => ({ count: state.count + 1 }))
}));

// Make Svelte-compatible
const store = {
  subscribe(callback) {
    return useStore.subscribe(callback);
  },
  ...useStore.getState()
};
```

## Built-in get() Helper

```javascript
import { get } from 'svelte/store';

// Get current value without subscribing
const currentValue = get(myStore);
```

📖 [Store contract documentation](https://svelte.dev/docs/svelte/stores)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*