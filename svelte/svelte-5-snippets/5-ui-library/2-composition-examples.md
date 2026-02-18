---
source_course: "svelte-5-snippets"
source_lesson: "svelte-5-snippets-composition-examples"
---

# Complete Component Implementations

Let's build some production-quality components using snippets.

## Example 1: Modal

```svelte
<!-- Modal.svelte -->
<script>
  let { 
    open = $bindable(false),
    title,
    children,
    footer,
    size = 'medium',
    closeOnBackdrop = true,
    closeOnEscape = true
  } = $props();
  
  function handleKeydown(e) {
    if (closeOnEscape && e.key === 'Escape') {
      open = false;
    }
  }
  
  function handleBackdrop() {
    if (closeOnBackdrop) {
      open = false;
    }
  }
</script>

<svelte:window onkeydown={handleKeydown} />

{#if open}
  <div class="modal-backdrop" onclick={handleBackdrop}>
    <div 
      class="modal modal-{size}" 
      onclick={(e) => e.stopPropagation()}
      role="dialog"
      aria-modal="true"
    >
      <header class="modal-header">
        <h2>{title}</h2>
        <button class="close" onclick={() => open = false}>×</button>
      </header>
      
      <div class="modal-body">
        {@render children?.()}
      </div>
      
      {#if footer}
        <footer class="modal-footer">
          {@render footer()}
        </footer>
      {/if}
    </div>
  </div>
{/if}
```

## Example 2: Data Table

```svelte
<!-- DataTable.svelte -->
<script>
  let {
    data,
    columns,
    row,
    empty,
    loading = false,
    sortable = false,
    onSort
  } = $props();
  
  let sortColumn = $state(null);
  let sortDirection = $state('asc');
  
  function handleSort(column) {
    if (sortColumn === column) {
      sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
    } else {
      sortColumn = column;
      sortDirection = 'asc';
    }
    onSort?.({ column, direction: sortDirection });
  }
</script>

<table class="data-table">
  <thead>
    <tr>
      {#each columns as column}
        <th 
          class:sortable
          class:sorted={sortColumn === column.key}
          onclick={() => sortable && handleSort(column.key)}
        >
          {column.label}
          {#if sortable && sortColumn === column.key}
            <span>{sortDirection === 'asc' ? '↑' : '↓'}</span>
          {/if}
        </th>
      {/each}
    </tr>
  </thead>
  <tbody>
    {#if loading}
      <tr>
        <td colspan={columns.length}>
          <div class="loading">Loading...</div>
        </td>
      </tr>
    {:else if data.length === 0}
      <tr>
        <td colspan={columns.length}>
          {#if empty}
            {@render empty()}
          {:else}
            <div class="empty">No data available</div>
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
<DataTable 
  data={users}
  columns={[
    { key: 'name', label: 'Name' },
    { key: 'email', label: 'Email' },
    { key: 'role', label: 'Role' }
  ]}
  sortable
  onSort={handleSort}
>
  {#snippet row(user, index)}
    <tr class:even={index % 2 === 0}>
      <td>{user.name}</td>
      <td>{user.email}</td>
      <td><Badge variant={user.role}>{user.role}</Badge></td>
    </tr>
  {/snippet}
  
  {#snippet empty()}
    <EmptyState>
      <p>No users found</p>
      <Button onclick={addUser}>Add First User</Button>
    </EmptyState>
  {/snippet}
</DataTable>
```

---

> 📘 *This lesson is part of the [Component Composition with Snippets](https://stanza.dev/courses/svelte-5-snippets) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*