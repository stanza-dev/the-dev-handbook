---
source_course: "svelte-5-sveltekit"
source_lesson: "svelte-5-sveltekit-progressive-enhancement"
---

# use:enhance: Forms Without Page Reloads

By default, forms cause full page reloads. `use:enhance` makes them work like SPAs while keeping the HTML form as a fallback.

## Basic Enhancement

```svelte
<script>
  import { enhance } from '$app/forms';
</script>

<form method="POST" use:enhance>
  <input name="email" type="email" />
  <button>Subscribe</button>
</form>
```

Now the form:
- Submits via fetch (no page reload)
- Automatically updates `form` prop with response
- Still works without JavaScript!

## Customizing Behavior

```svelte
<script>
  import { enhance } from '$app/forms';
  
  let isSubmitting = $state(false);
</script>

<form method="POST" use:enhance={() => {
  isSubmitting = true;
  
  return async ({ result, update }) => {
    isSubmitting = false;
    
    if (result.type === 'success') {
      // Custom success handling
      showToast('Saved!');
    }
    
    // Call update() to run default behavior
    // (update form prop, invalidate data)
    await update();
  };
}}>
  <button disabled={isSubmitting}>
    {isSubmitting ? 'Saving...' : 'Save'}
  </button>
</form>
```

## Result Types

```javascript
return async ({ result }) => {
  switch (result.type) {
    case 'success':
      // Action returned data
      console.log(result.data);
      break;
    case 'failure':
      // Action returned fail()
      console.log(result.data);
      break;
    case 'redirect':
      // Action threw redirect()
      console.log(result.location);
      break;
    case 'error':
      // Action threw error()
      console.log(result.error);
      break;
  }
};
```

## Preventing Default Update

```svelte
<form method="POST" use:enhance={() => {
  return async ({ result }) => {
    // Don't call update() - handle everything manually
    if (result.type === 'redirect') {
      goto(result.location);
    }
  };
}}>
```

## Reset Form Option

```svelte
<form method="POST" use:enhance={() => {
  return async ({ update }) => {
    await update({ reset: true }); // Reset form fields
  };
}}>
```

📖 [Progressive enhancement](https://svelte.dev/docs/kit/form-actions#Progressive-enhancement)

---

> 📘 *This lesson is part of the [SvelteKit 2 & Svelte 5: The Perfect Duo](https://stanza.dev/courses/svelte-5-sveltekit) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*