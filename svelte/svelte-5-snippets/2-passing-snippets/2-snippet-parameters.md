---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-snippet-parameters"
---

# Snippet Parameters: The Power Feature

Snippets can receive data from the component that renders them. This is incredibly powerful for lists, tables, and customizable components.

## Basic Data Passing

```svelte
<!-- List.svelte -->
<script>
  let { items, renderItem } = $props();
</script>

<ul>
  {#each items as item, index}
    <li>
      {@render renderItem(item, index)}
    </li>
  {/each}
</ul>
```

```svelte
<!-- Usage -->
<List items={products}>
  {#snippet renderItem(product, index)}
    <span class="number">{index + 1}.</span>
    <span class="name">{product.name}</span>
    <span class="price">${product.price}</span>
  {/snippet}
</List>
```

## Multiple Parameters

```svelte
<!-- Accordion.svelte -->
<script>
  let { items, header, content } = $props();
  let openIndex = $state(null);
</script>

{#each items as item, index}
  <div class="accordion-item">
    <button 
      class="accordion-header"
      onclick={() => openIndex = openIndex === index ? null : index}
    >
      {@render header(item, index === openIndex)}
    </button>
    
    {#if openIndex === index}
      <div class="accordion-content">
        {@render content(item)}
      </div>
    {/if}
  </div>
{/each}
```

```svelte
<!-- Usage -->
<Accordion items={faqs}>
  {#snippet header(faq, isOpen)}
    <span>{faq.question}</span>
    <span class="icon">{isOpen ? '−' : '+'}</span>
  {/snippet}
  
  {#snippet content(faq)}
    <p>{faq.answer}</p>
    {#if faq.links}
      <ul>
        {#each faq.links as link}
          <li><a href={link.url}>{link.text}</a></li>
        {/each}
      </ul>
    {/if}
  {/snippet}
</Accordion>
```

## Context + Actions

Pass functions and state to snippets:

```svelte
<!-- SelectList.svelte -->
<script>
  let { items, renderItem, onselect } = $props();
  let selectedId = $state(null);
  
  function select(item) {
    selectedId = item.id;
    onselect?.(item);
  }
</script>

<ul>
  {#each items as item}
    <li class:selected={selectedId === item.id}>
      {@render renderItem(item, item.id === selectedId, () => select(item))}
    </li>
  {/each}
</ul>
```

```svelte
<!-- Usage -->
<SelectList items={options} onselect={handleSelect}>
  {#snippet renderItem(option, isSelected, select)}
    <button onclick={select}>
      {#if isSelected}✓{/if}
      {option.label}
    </button>
  {/snippet}
</SelectList>
```

## TypeScript Type Safety

```svelte
<script lang="ts">
  import type { Snippet } from 'svelte';
  
  interface Props {
    items: User[];
    renderItem: Snippet<[user: User, index: number]>;
    header?: Snippet;
  }
  
  let { items, renderItem, header }: Props = $props();
</script>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*