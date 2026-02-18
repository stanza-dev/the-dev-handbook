---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-animate-directive"
---

# Animating List Changes

When items in a list are reordered, deleted, or inserted, Svelte can animate them to their new positions.

## Basic Usage

```svelte
<script>
  import { flip } from 'svelte/animate';
  
  let items = $state([1, 2, 3, 4, 5]);
  
  function shuffle() {
    items = items.sort(() => Math.random() - 0.5);
  }
</script>

<button onclick={shuffle}>Shuffle</button>

<ul>
  {#each items as item (item)}
    <li animate:flip>
      Item {item}
    </li>
  {/each}
</ul>
```

## How FLIP Works

FLIP stands for: **F**irst, **L**ast, **I**nvert, **P**lay

```
1. FIRST:  Record current positions
2. LAST:   Apply DOM changes, record new positions
3. INVERT: Calculate differences, apply inverse transform
4. PLAY:   Animate transform to zero (items move to new spots)
```

Svelte handles all of this automatically!

## Configuration

```svelte
{#each items as item (item)}
  <li animate:flip={{ duration: 300, easing: quintOut }}>
    {item}
  </li>
{/each}
```

## Important: Keys Are Required!

```svelte
<!-- ❌ Won't work without keys -->
{#each items as item}
  <li animate:flip>{item}</li>
{/each}

<!-- ✅ Must have unique keys -->
{#each items as item (item.id)}
  <li animate:flip>{item.name}</li>
{/each}
```

Without keys, Svelte can't track which DOM node belongs to which item.

## Combining with Transitions

```svelte
{#each items as item (item.id)}
  <li 
    animate:flip={{ duration: 200 }}
    in:fade={{ duration: 200 }}
    out:fade={{ duration: 200 }}
  >
    {item.name}
  </li>
{/each}
```

Now items:
- Fade in when added
- Animate position when reordered
- Fade out when removed

📖 [Animate directive](https://svelte.dev/docs/svelte/animate)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*