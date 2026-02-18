---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-drag-interactions"
---

# Building Drag and Drop

Drag and drop adds powerful interactivity to your apps. Let's implement it properly using native events.

## Basic Draggable Element

```svelte
<script>
  let position = $state({ x: 0, y: 0 });
  let isDragging = $state(false);
  let offset = { x: 0, y: 0 };
  
  function handleMouseDown(event) {
    isDragging = true;
    offset = {
      x: event.clientX - position.x,
      y: event.clientY - position.y
    };
  }
  
  function handleMouseMove(event) {
    if (!isDragging) return;
    position = {
      x: event.clientX - offset.x,
      y: event.clientY - offset.y
    };
  }
  
  function handleMouseUp() {
    isDragging = false;
  }
</script>

<svelte:window onmousemove={handleMouseMove} onmouseup={handleMouseUp} />

<div
  class="draggable"
  class:dragging={isDragging}
  style="transform: translate({position.x}px, {position.y}px)"
  onmousedown={handleMouseDown}
>
  Drag me!
</div>
```

## HTML5 Drag and Drop API

For more complex scenarios, use the native Drag and Drop API:

```svelte
<script>
  let items = $state(['Item 1', 'Item 2', 'Item 3']);
  let draggedIndex = $state(null);
  
  function handleDragStart(event, index) {
    draggedIndex = index;
    event.dataTransfer.effectAllowed = 'move';
  }
  
  function handleDragOver(event, index) {
    event.preventDefault();
    event.dataTransfer.dropEffect = 'move';
  }
  
  function handleDrop(event, dropIndex) {
    event.preventDefault();
    if (draggedIndex === null || draggedIndex === dropIndex) return;
    
    // Reorder array
    const newItems = [...items];
    const [removed] = newItems.splice(draggedIndex, 1);
    newItems.splice(dropIndex, 0, removed);
    items = newItems;
    draggedIndex = null;
  }
</script>

<ul>
  {#each items as item, i}
    <li
      draggable="true"
      ondragstart={(e) => handleDragStart(e, i)}
      ondragover={(e) => handleDragOver(e, i)}
      ondrop={(e) => handleDrop(e, i)}
      class:dragging={draggedIndex === i}
    >
      {item}
    </li>
  {/each}
</ul>
```

## Drag Events Reference

| Event | Fires When |
|-------|------------|
| `ondragstart` | User starts dragging |
| `ondrag` | Element is being dragged |
| `ondragend` | Drag operation ends |
| `ondragenter` | Enters a valid drop target |
| `ondragover` | Over a valid drop target |
| `ondragleave` | Leaves a drop target |
| `ondrop` | Dropped on a target |

## Touch Support

For mobile, add touch events:

```svelte
<div
  onmousedown={handleStart}
  ontouchstart={handleStart}
  ontouchmove={handleMove}
  ontouchend={handleEnd}
>
```

📖 [Event handling documentation](https://svelte.dev/docs/svelte/basic-markup)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*