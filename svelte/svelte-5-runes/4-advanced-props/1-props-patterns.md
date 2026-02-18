---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-props-patterns"
---

# Beyond Basic Props

Let's explore advanced patterns for working with component props.

## Destructuring with Defaults

```svelte
<script>
  let {
    // Required props (no default)
    id,
    label,
    
    // Optional props with defaults
    variant = 'primary',
    size = 'medium',
    disabled = false,
    
    // Callback props
    onclick = () => {},
    
    // Rest props for pass-through
    ...rest
  } = $props();
</script>
```

## The Rest Pattern

Capture "everything else" with rest props:

```svelte
<!-- Button.svelte -->
<script>
  let { children, variant = 'primary', ...rest } = $props();
</script>

<button class="btn btn-{variant}" {...rest}>
  {@render children()}
</button>
```

Usage:

```svelte
<Button 
  variant="danger" 
  disabled 
  aria-label="Delete item"
  onclick={handleDelete}
>
  Delete
</Button>
```

All extra props (`disabled`, `aria-label`, `onclick`) are spread onto `<button>`.

## TypeScript Props

```svelte
<script lang="ts">
  type ButtonVariant = 'primary' | 'secondary' | 'danger';
  type ButtonSize = 'small' | 'medium' | 'large';
  
  interface Props {
    label: string;
    variant?: ButtonVariant;
    size?: ButtonSize;
    disabled?: boolean;
    onclick?: () => void;
  }
  
  let {
    label,
    variant = 'primary',
    size = 'medium',
    disabled = false,
    onclick
  }: Props = $props();
</script>
```

## Renaming Props

Use aliasing when prop names conflict with reserved words:

```svelte
<script>
  let { 
    class: className = '',  // Rename 'class' to 'className'
    for: htmlFor            // Rename 'for' to 'htmlFor'
  } = $props();
</script>

<div class={className}>
  <label for={htmlFor}>...</label>
</div>
```

## Reactive Props

Props are reactive by default — changes from parent update the child:

```svelte
<script>
  let { count } = $props();
  
  // This is DERIVED from the prop
  let doubled = $derived(count * 2);
</script>

<p>Count: {count}, Doubled: {doubled}</p>
```

## $props vs $state for Local Copies

If you need to modify a prop locally without affecting the parent:

```svelte
<script>
  let { initialValue = 0 } = $props();
  
  // Create local state initialized from prop
  let localValue = $state(initialValue);
</script>
```

## Resources

- [$props Documentation](https://svelte.dev/docs/svelte/$props) — Complete reference for component props.

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*