---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-component-communication"
---

# Choosing the Right Pattern

Svelte 5 offers multiple ways for components to communicate. Let's understand when to use each.

## 1. Props (Parent → Child)

The simplest pattern — pass data down:

```svelte
<!-- Parent -->
<UserCard name={user.name} avatar={user.avatar} />

<!-- Child -->
<script>
  let { name, avatar } = $props();
</script>
```

## 2. Callback Props (Child → Parent)

Pass functions down, child calls them to communicate up:

```svelte
<!-- Parent -->
<script>
  function handleDelete(id) {
    items = items.filter(i => i.id !== id);
  }
</script>

<ItemList items={items} ondelete={handleDelete} />

<!-- Child -->
<script>
  let { items, ondelete } = $props();
</script>

{#each items as item}
  <li>
    {item.name}
    <button onclick={() => ondelete(item.id)}>Delete</button>
  </li>
{/each}
```

## 3. Bindable Props (Two-Way)

When parent and child both need to read/write:

```svelte
<!-- Parent -->
<Modal bind:open={showModal}>
  <p>Modal content</p>
</Modal>

<!-- Child -->
<script>
  let { open = $bindable(false), children } = $props();
</script>

{#if open}
  <div class="modal">
    {@render children()}
    <button onclick={() => open = false}>Close</button>
  </div>
{/if}
```

## 4. Context (Deep Passing)

When data needs to skip intermediate components:

```svelte
<!-- App.svelte -->
<script>
  import { setContext } from 'svelte';
  setContext('theme', { mode: 'dark' });
</script>

<!-- DeepChild.svelte (many levels down) -->
<script>
  import { getContext } from 'svelte';
  const theme = getContext('theme');
</script>
```

## 5. Shared State (.svelte.js)

For truly global state:

```js
// store.svelte.js
export const notifications = $state([]);

export function addNotification(message) {
  notifications.push({ id: Date.now(), message });
}
```

## Decision Tree

```
Does data flow DOWN only?
  → Props

Does child need to NOTIFY parent?
  → Callback props

Do both need to READ and WRITE?
  → $bindable

Is there DEEP nesting?
  → Context

Is state truly GLOBAL?
  → .svelte.js module
```

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*