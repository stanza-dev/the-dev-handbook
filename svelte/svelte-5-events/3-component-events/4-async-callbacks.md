---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-async-callbacks"
---

# Handling Async Operations in Callbacks

Many event handlers need to perform async operations like API calls. Let's explore patterns for handling async callbacks elegantly.

## Basic Async Callback

```svelte
<!-- Child.svelte -->
<script>
  let { onsave } = $props();
  let isSaving = $state(false);
  
  async function handleSave() {
    isSaving = true;
    try {
      await onsave(formData);
    } finally {
      isSaving = false;
    }
  }
</script>

<button onclick={handleSave} disabled={isSaving}>
  {isSaving ? 'Saving...' : 'Save'}
</button>
```

```svelte
<!-- Parent.svelte -->
<script>
  async function saveToApi(data) {
    const response = await fetch('/api/save', {
      method: 'POST',
      body: JSON.stringify(data)
    });
    if (!response.ok) throw new Error('Save failed');
  }
</script>

<Child onsave={saveToApi} />
```

## Handling Errors

The child can handle errors from async callbacks:

```svelte
<!-- Child.svelte -->
<script>
  let { onsave } = $props();
  let isSaving = $state(false);
  let error = $state(null);
  
  async function handleSave() {
    isSaving = true;
    error = null;
    
    try {
      await onsave(formData);
    } catch (e) {
      error = e.message;
    } finally {
      isSaving = false;
    }
  }
</script>

{#if error}
  <div class="error">{error}</div>
{/if}

<button onclick={handleSave} disabled={isSaving}>
  {isSaving ? 'Saving...' : 'Save'}
</button>
```

## Returning Data from Callbacks

Callbacks can return values:

```svelte
<!-- Child.svelte -->
<script>
  let { onvalidate } = $props();
  
  async function handleSubmit() {
    const isValid = await onvalidate(formData);
    if (isValid) {
      // Proceed with submission
    }
  }
</script>
```

```svelte
<!-- Parent.svelte -->
<Child onvalidate={async (data) => {
  const response = await fetch('/api/validate', { body: JSON.stringify(data) });
  return response.ok;
}} />
```

## TypeScript for Async Callbacks

```svelte
<script lang="ts">
  type Props = {
    onsave: (data: FormData) => Promise<void>;
    onvalidate?: (data: FormData) => Promise<boolean>;
    oncancel?: () => void; // Sync callback
  };
  
  let { onsave, onvalidate, oncancel }: Props = $props();
</script>
```

📖 [Props documentation](https://svelte.dev/docs/svelte/$props)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*