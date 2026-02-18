---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-callback-props"
---

# The New Way: Callback Props

In Svelte 4, we used `createEventDispatcher` to send events from child to parent. Svelte 5 deprecates this in favor of simple callback props - plain JavaScript functions passed as props.

## The Old Way (Svelte 4) - Don't Use

```svelte
<!-- Child.svelte - OLD WAY -->
<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();
  
  function handleClick() {
    dispatch('delete', { id: 123 });
  }
</script>

<button on:click={handleClick}>Delete</button>
```

```svelte
<!-- Parent.svelte - OLD WAY -->
<Child on:delete={(e) => removeItem(e.detail.id)} />
```

## The New Way (Svelte 5) - Use This!

```svelte
<!-- Child.svelte - NEW WAY -->
<script>
  let { ondelete } = $props();
</script>

<button onclick={() => ondelete(123)}>Delete</button>
```

```svelte
<!-- Parent.svelte - NEW WAY -->
<script>
  import Child from './Child.svelte';
  
  function removeItem(id) {
    items = items.filter(item => item.id !== id);
  }
</script>

<Child ondelete={removeItem} />
```

## Why Is This Better?

**1. Just JavaScript**
No Svelte-specific API to learn. It's just passing functions as props - a pattern used in React, Vue, and vanilla JS.

**2. Perfect TypeScript Support**
```svelte
<script lang="ts">
  type Props = {
    ondelete: (id: number) => void;
    onsave?: (data: FormData) => Promise<void>;
  };
  
  let { ondelete, onsave }: Props = $props();
</script>
```

**3. Optional Callbacks with Defaults**
```svelte
<script>
  let { 
    ondelete = () => {},  // No-op default
    onsave                 // Required prop
  } = $props();
</script>
```

**4. Direct Data Passing**
No wrapping in `detail` - just pass the data directly:
```svelte
<!-- Child -->
<button onclick={() => onselect({ id: item.id, name: item.name })}>
  Select
</button>

<!-- Parent -->
<Child onselect={(item) => selectedItem = item} />
```

## Naming Convention

By convention, callback props start with `on` followed by the action name:
- `onclick` - for click events
- `ondelete` - for delete actions
- `onsave` - for save actions
- `onchange` - for change events
- `onsubmit` - for form submissions

This makes it clear these props are event callbacks.

📖 [Props documentation](https://svelte.dev/docs/svelte/$props)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*