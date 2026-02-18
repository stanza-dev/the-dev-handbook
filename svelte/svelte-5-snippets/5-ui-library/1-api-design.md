---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-api-design"
---

# Creating Developer-Friendly Components

When building reusable components, the API design matters as much as the implementation.

## Principles

### 1. Progressive Disclosure

Start simple, add complexity as needed:

```svelte
<!-- Simple usage -->
<Button>Click me</Button>

<!-- With options -->
<Button variant="primary" size="large">Submit</Button>

<!-- Fully customized -->
<Button 
  variant="primary" 
  size="large"
  disabled={loading}
  onclick={handleSubmit}
>
  {#snippet prefix()}<Spinner />{/snippet}
  Submitting...
</Button>
```

### 2. Sensible Defaults

```svelte
<script>
  let {
    variant = 'primary',     // Good default
    size = 'medium',         // Good default
    disabled = false,        // Good default
    type = 'button',         // Safe default (not 'submit')
    children,
    ...rest
  } = $props();
</script>
```

### 3. Flexibility Without Complexity

```svelte
<!-- Alert.svelte -->
<script>
  let {
    // Core props
    type = 'info',
    title,
    children,
    
    // Optional customization
    icon,
    dismissible = false,
    ondismiss,
    
    // Pass-through
    ...rest
  } = $props();
  
  const defaultIcons = {
    info: 'ℹ️',
    success: '✅',
    warning: '⚠️',
    error: '❌'
  };
</script>

<div class="alert alert-{type}" role="alert" {...rest}>
  <span class="alert-icon">
    {#if icon}
      {@render icon()}
    {:else}
      {defaultIcons[type]}
    {/if}
  </span>
  
  <div class="alert-content">
    {#if title}
      <strong class="alert-title">{title}</strong>
    {/if}
    {@render children?.()}
  </div>
  
  {#if dismissible}
    <button class="alert-dismiss" onclick={ondismiss}>×</button>
  {/if}
</div>
```

## TypeScript for Better DX

```svelte
<script lang="ts">
  import type { Snippet } from 'svelte';
  import type { HTMLButtonAttributes } from 'svelte/elements';
  
  type Variant = 'primary' | 'secondary' | 'danger' | 'ghost';
  type Size = 'small' | 'medium' | 'large';
  
  interface Props extends HTMLButtonAttributes {
    variant?: Variant;
    size?: Size;
    loading?: boolean;
    prefix?: Snippet;
    suffix?: Snippet;
    children: Snippet;
  }
  
  let {
    variant = 'primary',
    size = 'medium',
    loading = false,
    disabled = false,
    prefix,
    suffix,
    children,
    ...rest
  }: Props = $props();
</script>
```

## Documentation Pattern

```svelte
<!--
@component
A customizable button component.

@example
```svelte
<Button variant="primary" onclick={handleClick}>
  Click me
</Button>
```

@prop {Variant} [variant='primary'] - Visual style
@prop {Size} [size='medium'] - Button size
@prop {boolean} [loading=false] - Show loading state
@prop {Snippet} [prefix] - Content before label
@prop {Snippet} [suffix] - Content after label
@prop {Snippet} children - Button label
-->
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*