---
source_course: "svelte-5-motion"
source_lesson: "svelte-5-motion-sortable-list"
---

# Practical: Drag-to-Reorder List

Let's build a draggable list with smooth animations.

## Complete Example

```svelte
<script>
  import { flip } from 'svelte/animate';
  import { fade } from 'svelte/transition';
  
  let items = $state([
    { id: 1, name: 'Learn Svelte' },
    { id: 2, name: 'Build an app' },
    { id: 3, name: 'Deploy to production' }
  ]);
  
  let draggingId = $state(null);
  
  function dragStart(event, id) {
    draggingId = id;
    event.dataTransfer.effectAllowed = 'move';
  }
  
  function dragOver(event, targetId) {
    event.preventDefault();
    if (draggingId === targetId) return;
    
    const dragIndex = items.findIndex(i => i.id === draggingId);
    const targetIndex = items.findIndex(i => i.id === targetId);
    
    // Reorder array
    const newItems = [...items];
    const [removed] = newItems.splice(dragIndex, 1);
    newItems.splice(targetIndex, 0, removed);
    items = newItems;
  }
  
  function dragEnd() {
    draggingId = null;
  }
  
  function removeItem(id) {
    items = items.filter(i => i.id !== id);
  }
</script>

<ul class="sortable">
  {#each items as item (item.id)}
    <li
      animate:flip={{ duration: 200 }}
      out:fade={{ duration: 150 }}
      draggable="true"
      class:dragging={draggingId === item.id}
      ondragstart={(e) => dragStart(e, item.id)}
      ondragover={(e) => dragOver(e, item.id)}
      ondragend={dragEnd}
    >
      <span class="handle">☰</span>
      <span class="text">{item.name}</span>
      <button onclick={() => removeItem(item.id)}>×</button>
    </li>
  {/each}
</ul>

<style>
  .sortable {
    list-style: none;
    padding: 0;
  }
  
  li {
    display: flex;
    align-items: center;
    padding: 12px;
    background: white;
    border: 1px solid #ddd;
    margin: 4px 0;
    cursor: grab;
  }
  
  li.dragging {
    opacity: 0.5;
  }
  
  .handle {
    margin-right: 12px;
    color: #999;
  }
  
  .text {
    flex: 1;
  }
</style>
```

## Key Techniques

1. **animate:flip** - Smoothly moves items to new positions
2. **out:fade** - Animates removed items
3. **draggable="true"** - Native HTML5 drag and drop
4. **Keys** - Essential for tracking items

📖 [HTML5 Drag and Drop](https://developer.mozilla.org/en-US/docs/Web/API/HTML_Drag_and_Drop_API)

---

> 📘 *This lesson is part of the [Motion & Transitions](https://stanza.dev/courses/svelte-5-motion) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*