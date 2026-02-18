---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-named-snippets"
---

# Passing Multiple Content Blocks

Components often need more than just `children`. Named snippets let you pass multiple content blocks.

## The Pattern

```svelte
<!-- Card.svelte -->
<script>
  let { header, children, footer } = $props();
</script>

<div class="card">
  {#if header}
    <div class="card-header">
      {@render header()}
    </div>
  {/if}
  
  <div class="card-body">
    {@render children?.()}
  </div>
  
  {#if footer}
    <div class="card-footer">
      {@render footer()}
    </div>
  {/if}
</div>
```

## Passing Named Snippets

```svelte
<Card>
  {#snippet header()}
    <h2>Card Title</h2>
    <span class="badge">New</span>
  {/snippet}
  
  <p>This is the main content.</p>
  <p>It can have multiple elements.</p>
  
  {#snippet footer()}
    <button>Cancel</button>
    <button>Save</button>
  {/snippet}
</Card>
```

Notice:
- Named snippets go **inside** the component tags
- Non-snippet content becomes `children`
- Order doesn't matter — named snippets are matched by name

## Explicit children Snippet

You can also define children explicitly:

```svelte
<Card>
  {#snippet header()}
    <h2>Title</h2>
  {/snippet}
  
  {#snippet children()}
    <p>Explicit body content</p>
  {/snippet}
  
  {#snippet footer()}
    <button>OK</button>
  {/snippet}
</Card>
```

## Real-World: Table Component

```svelte
<!-- DataTable.svelte -->
<script>
  let { data, header, row, empty } = $props();
</script>

<table>
  <thead>
    {@render header()}
  </thead>
  <tbody>
    {#if data.length === 0}
      <tr>
        <td colspan="100">
          {#if empty}
            {@render empty()}
          {:else}
            No data available
          {/if}
        </td>
      </tr>
    {:else}
      {#each data as item, index}
        {@render row(item, index)}
      {/each}
    {/if}
  </tbody>
</table>
```

```svelte
<!-- Usage -->
<DataTable data={users}>
  {#snippet header()}
    <tr>
      <th>Name</th>
      <th>Email</th>
      <th>Actions</th>
    </tr>
  {/snippet}
  
  {#snippet row(user, index)}
    <tr class:even={index % 2 === 0}>
      <td>{user.name}</td>
      <td>{user.email}</td>
      <td>
        <button onclick={() => edit(user)}>Edit</button>
      </td>
    </tr>
  {/snippet}
  
  {#snippet empty()}
    <div class="empty-state">
      <img src="/empty.svg" alt="" />
      <p>No users found</p>
      <button onclick={addUser}>Add First User</button>
    </div>
  {/snippet}
</DataTable>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*