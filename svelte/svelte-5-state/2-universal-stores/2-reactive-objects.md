---
source_course: "svelte-5-state"
source_lesson: "svelte-5-state-reactive-objects"
---

# Creating Feature-Rich Stores

Let's build practical reactive stores using the `.svelte.js` pattern.

## Pattern 1: Simple Object Store

```javascript
// user.svelte.js
export const user = $state({
  name: '',
  email: '',
  isAuthenticated: false
});

export function login(name, email) {
  user.name = name;
  user.email = email;
  user.isAuthenticated = true;
}

export function logout() {
  user.name = '';
  user.email = '';
  user.isAuthenticated = false;
}
```

## Pattern 2: Store with Derived Values

```javascript
// cart.svelte.js
export const cart = $state({
  items: [],
  
  add(product, quantity = 1) {
    const existing = this.items.find(i => i.product.id === product.id);
    if (existing) {
      existing.quantity += quantity;
    } else {
      this.items.push({ product, quantity });
    }
  },
  
  remove(productId) {
    this.items = this.items.filter(i => i.product.id !== productId);
  },
  
  clear() {
    this.items = [];
  }
});

// Derived values
export const itemCount = $derived(
  cart.items.reduce((sum, item) => sum + item.quantity, 0)
);

export const total = $derived(
  cart.items.reduce((sum, item) => sum + item.product.price * item.quantity, 0)
);
```

## Pattern 3: Store Factory Function

For reusable store logic:

```javascript
// createTodoStore.svelte.js
export function createTodoStore(initialTodos = []) {
  const todos = $state(initialTodos);
  
  return {
    get items() { return todos; },
    
    get remaining() {
      return todos.filter(t => !t.completed).length;
    },
    
    add(text) {
      todos.push({
        id: crypto.randomUUID(),
        text,
        completed: false
      });
    },
    
    toggle(id) {
      const todo = todos.find(t => t.id === id);
      if (todo) todo.completed = !todo.completed;
    },
    
    remove(id) {
      const index = todos.findIndex(t => t.id === id);
      if (index !== -1) todos.splice(index, 1);
    }
  };
}
```

```svelte
<script>
  import { createTodoStore } from './createTodoStore.svelte.js';
  
  const todos = createTodoStore([
    { id: '1', text: 'Learn Svelte', completed: true }
  ]);
</script>

{#each todos.items as todo}
  <div>
    <input type="checkbox" checked={todo.completed} onchange={() => todos.toggle(todo.id)} />
    {todo.text}
  </div>
{/each}

<p>{todos.remaining} remaining</p>
```

📖 [State documentation](https://svelte.dev/docs/svelte/$state)

---

> 📘 *This lesson is part of the [Scalable State Management](https://stanza.dev/courses/svelte-5-state) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*