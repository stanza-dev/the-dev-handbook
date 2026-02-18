---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-props-migration"
---

# Props Migration

Svelte 5 uses `$props()` instead of individual `export let` declarations.

## Basic Props

```svelte
<!-- Svelte 4 -->
<script>
  export let name;
  export let age = 25; // Default value
</script>

<!-- Svelte 5 -->
<script>
  let { name, age = 25 } = $props();
</script>
```

## Multiple Props

```svelte
<!-- Svelte 4 -->
<script>
  export let title;
  export let description;
  export let isActive = false;
  export let count = 0;
</script>

<!-- Svelte 5 -->
<script>
  let { 
    title, 
    description, 
    isActive = false, 
    count = 0 
  } = $props();
</script>
```

## Rest Props ($$restProps Replacement)

```svelte
<!-- Svelte 4 -->
<script>
  export let type = 'button';
</script>

<button {type} {...$$restProps}>
  <slot />
</button>

<!-- Svelte 5 -->
<script>
  let { type = 'button', children, ...rest } = $props();
</script>

<button {type} {...rest}>
  {@render children()}
</button>
```

## Renaming Props

```svelte
<!-- Svelte 4 -->
<script>
  let className;
  export { className as class };
</script>

<!-- Svelte 5 -->
<script>
  let { class: className } = $props();
</script>
```

## TypeScript Props

```svelte
<!-- Svelte 4 -->
<script lang="ts">
  export let name: string;
  export let count: number = 0;
</script>

<!-- Svelte 5 -->
<script lang="ts">
  type Props = {
    name: string;
    count?: number;
  };
  
  let { name, count = 0 }: Props = $props();
</script>
```

📖 [$props documentation](https://svelte.dev/docs/svelte/$props)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*