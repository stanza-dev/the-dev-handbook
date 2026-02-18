---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-recursive-snippets"
---

# Snippets That Call Themselves

One of the most powerful snippet features is **recursion** — a snippet can render itself!

## Basic Recursion: Tree Structure

```svelte
<script>
  let fileTree = $state({
    name: 'src',
    children: [
      { name: 'app', children: [
        { name: 'page.svelte' },
        { name: 'layout.svelte' }
      ]},
      { name: 'lib', children: [
        { name: 'utils.js' },
        { name: 'components', children: [
          { name: 'Button.svelte' },
          { name: 'Card.svelte' }
        ]}
      ]}
    ]
  });
</script>

{#snippet treeNode(node, depth = 0)}
  <div style="padding-left: {depth * 20}px">
    <span class="name">
      {node.children ? '📁' : '📄'} {node.name}
    </span>
    
    {#if node.children}
      {#each node.children as child}
        {@render treeNode(child, depth + 1)}  <!-- Recursion! -->
      {/each}
    {/if}
  </div>
{/snippet}

<div class="file-tree">
  {@render treeNode(fileTree)}
</div>
```

## Expandable Tree

```svelte
<script>
  let expanded = $state(new Set());
  
  function toggle(path) {
    if (expanded.has(path)) {
      expanded.delete(path);
    } else {
      expanded.add(path);
    }
    expanded = expanded;  // Trigger update
  }
</script>

{#snippet node(item, path = '')}
  {@const currentPath = `${path}/${item.name}`}
  {@const isExpanded = expanded.has(currentPath)}
  {@const hasChildren = item.children?.length > 0}
  
  <div class="node">
    <button 
      class="toggle"
      onclick={() => hasChildren && toggle(currentPath)}
      disabled={!hasChildren}
    >
      {#if hasChildren}
        {isExpanded ? '▼' : '▶'}
      {:else}
        •
      {/if}
    </button>
    <span class="label">{item.name}</span>
  </div>
  
  {#if hasChildren && isExpanded}
    <div class="children">
      {#each item.children as child}
        {@render node(child, currentPath)}
      {/each}
    </div>
  {/if}
{/snippet}

{@render node(data)}
```

## Comment Thread Example

```svelte
{#snippet comment(c, depth = 0)}
  <article class="comment" style="--depth: {depth}">
    <header>
      <img src={c.author.avatar} alt="" />
      <strong>{c.author.name}</strong>
      <time>{c.date}</time>
    </header>
    <p>{c.text}</p>
    <button onclick={() => reply(c.id)}>Reply</button>
    
    {#if c.replies?.length}
      <div class="replies">
        {#each c.replies as reply}
          {@render comment(reply, depth + 1)}  <!-- Nested! -->
        {/each}
      </div>
    {/if}
  </article>
{/snippet}

{#each comments as c}
  {@render comment(c)}
{/each}
```

## Base Case is Essential

Like any recursion, you need a termination condition:

```svelte
{#snippet countDown(n)}
  {#if n <= 0}    <!-- Base case -->
    <p>Done!</p>
  {:else}
    <p>{n}...</p>
    {@render countDown(n - 1)}  <!-- Recursive case -->
  {/if}
{/snippet}
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*