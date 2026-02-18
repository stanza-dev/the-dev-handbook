---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-transition-modes"
---

# Coordinating Multiple Transitions

When elements are swapped (one leaves, one enters), you might want to control the order.

## The Problem: Overlapping Transitions

```svelte
<script>
  let current = $state('a');
</script>

{#if current === 'a'}
  <div transition:fly={{ x: -100 }}>A</div>
{:else}
  <div transition:fly={{ x: 100 }}>B</div>
{/if}
```

Both transitions run simultaneously, which can look chaotic.

## Using Key Blocks

The `{#key}` block forces elements to be recreated:

```svelte
{#key current}
  <div transition:fade>
    {current}
  </div>
{/key}
```

Every time `current` changes, the element transitions out, then a new one transitions in.

## CSS-Based Sequencing

Use delays to sequence animations:

```svelte
<script>
  let current = $state('a');
  const duration = 300;
</script>

{#if current === 'a'}
  <div 
    in:fly={{ x: -100, duration, delay: duration }}
    out:fly={{ x: 100, duration }}
  >
    A
  </div>
{:else}
  <div 
    in:fly={{ x: -100, duration, delay: duration }}
    out:fly={{ x: 100, duration }}
  >
    B
  </div>
{/if}
```

The entering element waits (`delay: duration`) for the leaving one to finish.

## Using Events for Complex Coordination

```svelte
<script>
  let transitioning = $state(false);
  
  function switchContent() {
    if (transitioning) return;
    current = current === 'a' ? 'b' : 'a';
  }
</script>

{#key current}
  <div 
    transition:fade
    onoutrostart={() => transitioning = true}
    onintroend={() => transitioning = false}
  >
    {current}
  </div>
{/key}

<button onclick={switchContent} disabled={transitioning}>
  Switch
</button>
```

📖 [Key blocks](https://svelte.dev/docs/svelte/key)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*