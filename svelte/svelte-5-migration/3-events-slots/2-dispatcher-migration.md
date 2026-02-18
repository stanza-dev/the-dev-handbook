---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-dispatcher-migration"
---

# Component Events to Callback Props

`createEventDispatcher` is deprecated. Use callback props instead.

## Basic Event Dispatch

```svelte
<!-- Svelte 4: Child -->
<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();
</script>

<button on:click={() => dispatch('submit', { id: 123 })}>
  Submit
</button>

<!-- Svelte 4: Parent -->
<Child on:submit={(e) => handleSubmit(e.detail)} />
```

```svelte
<!-- Svelte 5: Child -->
<script>
  let { onsubmit } = $props();
</script>

<button onclick={() => onsubmit?.({ id: 123 })}>
  Submit
</button>

<!-- Svelte 5: Parent -->
<Child onsubmit={(data) => handleSubmit(data)} />
```

## Multiple Events

```svelte
<!-- Svelte 4: Child -->
<script>
  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();
  
  function save() { dispatch('save'); }
  function delete() { dispatch('delete', { id }); }
  function cancel() { dispatch('cancel'); }
</script>
```

```svelte
<!-- Svelte 5: Child -->
<script>
  let { onsave, ondelete, oncancel } = $props();
</script>

<button onclick={() => onsave?.()}>Save</button>
<button onclick={() => ondelete?.({ id })}>Delete</button>
<button onclick={() => oncancel?.()}>Cancel</button>
```

## Event with Data

```svelte
<!-- Svelte 4 -->
dispatch('select', { item, index });
// Parent: on:select={(e) => use(e.detail.item, e.detail.index)}

<!-- Svelte 5 -->
onselect?.({ item, index });
// Parent: onselect={({ item, index }) => use(item, index)}
```

The data is passed directly, not wrapped in `detail`!

## Optional vs Required Events

```svelte
<script>
  // Optional - use ?.() to safely call
  let { onsave } = $props();
  
  // Or provide default
  let { oncancel = () => {} } = $props();
</script>

<button onclick={() => onsave?.()}>Save</button>
<button onclick={oncancel}>Cancel</button>
```

📖 [Component events](https://svelte.dev/docs/svelte/v5-migration-guide#Event-changes)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*