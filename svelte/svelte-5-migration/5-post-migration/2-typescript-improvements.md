---
source_course: "svelte-5-migration"
source_lesson: "svelte-5-migration-typescript-improvements"
---

# Better TypeScript with Runes

Svelte 5 has significantly better TypeScript support. Take advantage of it!

## Typed Props

```svelte
<script lang="ts">
  type Props = {
    name: string;
    count?: number;
    onsubmit: (data: FormData) => void;
    children: import('svelte').Snippet;
  };
  
  let { name, count = 0, onsubmit, children }: Props = $props();
</script>
```

## Typed State

```svelte
<script lang="ts">
  type User = {
    id: number;
    name: string;
    email: string;
  };
  
  let user = $state<User | null>(null);
  let users = $state<User[]>([]);
</script>
```

## Typed Derived

```svelte
<script lang="ts">
  let items = $state<Item[]>([]);
  
  // Type is inferred!
  let activeItems = $derived(items.filter(i => i.active));
  // activeItems: Item[]
</script>
```

## Typed Effects

```svelte
<script lang="ts">
  let count = $state(0);
  
  // TypeScript knows count is number
  $effect(() => {
    const doubled = count * 2; // No error
    console.log(doubled);
  });
</script>
```

## Component Typing

```svelte
<script lang="ts">
  import type { HTMLAttributes } from 'svelte/elements';
  
  type Props = HTMLAttributes<HTMLButtonElement> & {
    variant?: 'primary' | 'secondary';
  };
  
  let { variant = 'primary', ...rest }: Props = $props();
</script>

<button class="btn-{variant}" {...rest} />
```

## Snippet Typing

```svelte
<script lang="ts">
  import type { Snippet } from 'svelte';
  
  type Props = {
    header: Snippet;
    row: Snippet<[item: Item, index: number]>;
    footer?: Snippet;
  };
  
  let { header, row, footer }: Props = $props();
</script>
```

📖 [TypeScript guide](https://svelte.dev/docs/svelte/typescript)

---

> 📘 *This lesson is part of the [Moving to Svelte 5](https://stanza.dev/courses/svelte-5-migration) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*