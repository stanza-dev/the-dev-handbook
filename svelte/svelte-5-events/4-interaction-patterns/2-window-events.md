---
source_course: "svelte-5-events"
source_lesson: "svelte-5-events-window-events"
---

# Global Events with svelte:window and svelte:document

Some events happen at the window or document level - like resize, scroll, or global keyboard shortcuts. Svelte provides special elements for these.

## svelte:window

Listen to events on the `window` object:

```svelte
<script>
  let innerWidth = $state(0);
  let innerHeight = $state(0);
  let scrollY = $state(0);
  
  function handleResize() {
    console.log(`Window: ${innerWidth}x${innerHeight}`);
  }
  
  function handleKeydown(event) {
    if (event.key === 'Escape') closeModal();
  }
</script>

<!-- Bind to window properties -->
<svelte:window 
  bind:innerWidth 
  bind:innerHeight
  bind:scrollY
  onresize={handleResize}
  onkeydown={handleKeydown}
/>

<p>Window size: {innerWidth} × {innerHeight}</p>
<p>Scroll position: {scrollY}px</p>
```

## Bindable Window Properties

You can bind to these `window` properties:
- `innerWidth`, `innerHeight` - viewport dimensions
- `outerWidth`, `outerHeight` - browser window dimensions
- `scrollX`, `scrollY` - scroll position (also writable!)
- `online` - navigator.onLine status
- `devicePixelRatio` - screen DPI

```svelte
<script>
  let scrollY = $state(0);
  
  function scrollToTop() {
    scrollY = 0; // Writing to scrollY scrolls the page!
  }
</script>

<svelte:window bind:scrollY />

{#if scrollY > 500}
  <button onclick={scrollToTop} class="back-to-top">
    ↑ Back to top
  </button>
{/if}
```

## svelte:document

Similar to `svelte:window` but for document-level events:

```svelte
<script>
  let isFullscreen = $state(false);
  
  function handleFullscreenChange() {
    isFullscreen = !!document.fullscreenElement;
  }
  
  function handleVisibilityChange() {
    if (document.hidden) {
      pauseVideo();
    } else {
      resumeVideo();
    }
  }
</script>

<svelte:document 
  onfullscreenchange={handleFullscreenChange}
  onvisibilitychange={handleVisibilityChange}
/>
```

## svelte:body

For events on the `<body>` element:

```svelte
<svelte:body onmouseenter={() => showCursor = true} />
```

## When to Use Which?

| Event | Use |
|-------|-----|
| `resize`, `scroll`, keyboard shortcuts | `<svelte:window>` |
| `visibilitychange`, `fullscreenchange` | `<svelte:document>` |
| `mouseenter`/`mouseleave` on body | `<svelte:body>` |

📖 [svelte:window documentation](https://svelte.dev/docs/svelte/svelte-window)

---

> 📘 *This lesson is part of the [Event Handling & User Interaction](https://stanza.dev/courses/svelte-5-events) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*