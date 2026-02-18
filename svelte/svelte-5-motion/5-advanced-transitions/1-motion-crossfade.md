---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-crossfade"
---

# The Crossfade Effect

`crossfade` creates the illusion of an element moving from one place to another - even between different components!

## How It Works

`crossfade()` returns a pair of transitions: `send` and `receive`.

```svelte
<script>
  import { crossfade } from 'svelte/transition';
  
  const [send, receive] = crossfade({
    duration: 400,
    fallback: fade // Used when no matching element
  });
  
  let todos = $state([...]);
  let done = $state([...]);
  
  function complete(todo) {
    todos = todos.filter(t => t.id !== todo.id);
    done = [...done, todo];
  }
</script>

<div class="todo-list">
  <h2>Todo</h2>
  {#each todos as todo (todo.id)}
    <div 
      in:receive={{ key: todo.id }}
      out:send={{ key: todo.id }}
      onclick={() => complete(todo)}
    >
      {todo.text}
    </div>
  {/each}
</div>

<div class="done-list">
  <h2>Done</h2>
  {#each done as item (item.id)}
    <div 
      in:receive={{ key: item.id }}
      out:send={{ key: item.id }}
    >
      {item.text}
    </div>
  {/each}
</div>
```

## The Magic: Matching Keys

When an element with a key is `send` from one place and `receive`d in another:

1. Element fades out from original position
2. "Ghost" animates to new position
3. Element fades in at new position

The key must match: `send={{ key: id }}` ↔ `receive={{ key: id }}`

## Configuration

```javascript
const [send, receive] = crossfade({
  duration: 400,
  delay: 0,
  easing: quintOut,
  fallback: fade // What to do when no matching element exists
});
```

## Real-World Example: Shopping Cart

```svelte
<script>
  const [send, receive] = crossfade({ fallback: scale });
  
  let products = $state([...]);
  let cart = $state([]);
</script>

<!-- Product flies to cart when added! -->
{#each products as product (product.id)}
  <div out:send={{ key: product.id }}>
    {product.name}
    <button onclick={() => addToCart(product)}>Add</button>
  </div>
{/each}

<div class="cart">
  {#each cart as item (item.id)}
    <div in:receive={{ key: item.id }}>
      {item.name}
    </div>
  {/each}
</div>
```

📖 [Crossfade documentation](https://svelte.dev/docs/svelte/transition#crossfade)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*