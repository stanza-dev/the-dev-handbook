---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-forwarding-events"
---

# Forwarding All Events with Spread

When building wrapper components, you often want to forward all event handlers to the underlying element. Svelte 5 makes this elegant with prop spreading.

## The Problem

Imagine you're building a custom Button component:

```svelte
<!-- Button.svelte -->
<script>
  let { children, variant = 'primary' } = $props();
</script>

<button class="btn btn-{variant}">
  {@render children()}
</button>
```

How do you let parent components add `onclick`, `onmouseenter`, `disabled`, etc.?

## The Solution: Spread Props

Capture remaining props with rest syntax and spread them:

```svelte
<!-- Button.svelte -->
<script>
  let { children, variant = 'primary', ...rest } = $props();
</script>

<button class="btn btn-{variant}" {...rest}>
  {@render children()}
</button>
```

Now parent can use it like a native button:

```svelte
<script>
  import Button from './Button.svelte';
</script>

<Button 
  variant="danger" 
  onclick={() => deleteItem()}
  disabled={isLoading}
  title="Delete this item"
>
  Delete
</Button>
```

## What Gets Forwarded?

Everything in `...rest`:
- Event handlers: `onclick`, `onmouseover`, `onkeydown`
- HTML attributes: `disabled`, `type`, `title`, `aria-label`
- Data attributes: `data-testid`, `data-id`
- Styles: `style`
- Classes: (though handle separately, see below)

## Combining Classes

Class handling needs special care:

```svelte
<script>
  let { class: className = '', variant = 'primary', ...rest } = $props();
</script>

<!-- Combine your classes with passed classes -->
<button class="btn btn-{variant} {className}" {...rest}>
  {@render children()}
</button>
```

Usage:
```svelte
<Button class="mt-4 w-full" variant="primary">
  Submit
</Button>
```

## Type-Safe Forwarding

```svelte
<script lang="ts">
  import type { HTMLButtonAttributes } from 'svelte/elements';
  
  type Props = HTMLButtonAttributes & {
    variant?: 'primary' | 'secondary' | 'danger';
  };
  
  let { variant = 'primary', ...rest }: Props = $props();
</script>

<button class="btn-{variant}" {...rest} />
```

Now TypeScript knows all valid button attributes!

📖 [Props documentation](https://svelte.dev/docs/svelte/$props)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*