---
source_course: "svelte-5-runes"
source_lesson: "svelte-5-runes-effect-patterns"
---

# Real-World Effect Patterns

Let's explore common patterns for using effects effectively.

## Pattern 1: Debounced API Calls

```js
let searchQuery = $state('');
let results = $state([]);

$effect(() => {
  const query = searchQuery;  // Capture current value
  if (!query) {
    results = [];
    return;
  }
  
  // Debounce: wait 300ms before searching
  const timeoutId = setTimeout(async () => {
    const response = await fetch(`/api/search?q=${query}`);
    const data = await response.json();
    
    // Only update if query hasn't changed
    if (searchQuery === query) {
      results = data;
    }
  }, 300);
  
  return () => clearTimeout(timeoutId);
});
```

## Pattern 2: WebSocket Connection

```js
let messages = $state([]);
let connected = $state(false);
let roomId = $state('general');

$effect(() => {
  const ws = new WebSocket(`wss://chat.example.com/rooms/${roomId}`);
  
  ws.onopen = () => connected = true;
  ws.onclose = () => connected = false;
  ws.onmessage = (event) => {
    messages = [...messages, JSON.parse(event.data)];
  };
  
  return () => {
    ws.close();
    connected = false;
  };
});
```

## Pattern 3: Canvas Drawing

```js
let width = $state(400);
let height = $state(300);
let color = $state('#ff0000');
let canvas;

$effect(() => {
  if (!canvas) return;
  
  const ctx = canvas.getContext('2d');
  ctx.clearRect(0, 0, width, height);
  ctx.fillStyle = color;
  ctx.fillRect(50, 50, 100, 100);
});
```

## Pattern 4: Media Query Listener

```js
let isMobile = $state(false);

$effect(() => {
  const mq = window.matchMedia('(max-width: 768px)');
  isMobile = mq.matches;
  
  function handler(e) {
    isMobile = e.matches;
  }
  
  mq.addEventListener('change', handler);
  return () => mq.removeEventListener('change', handler);
});
```

## Pattern 5: Intersection Observer

```js
let element;
let isVisible = $state(false);

$effect(() => {
  if (!element) return;
  
  const observer = new IntersectionObserver(([entry]) => {
    isVisible = entry.isIntersecting;
  });
  
  observer.observe(element);
  
  return () => observer.disconnect();
});
```

## Pattern 6: Third-Party Library Integration

```js
let chartElement;
let data = $state([]);
let chart = null;

$effect(() => {
  if (!chartElement) return;
  
  // Create chart on first run
  if (!chart) {
    chart = new Chart(chartElement, {
      type: 'line',
      data: { datasets: [] }
    });
  }
  
  // Update data
  chart.data.datasets = data;
  chart.update();
  
  return () => {
    chart?.destroy();
    chart = null;
  };
});
```

---

> 📘 *This lesson is part of the [Mastering Runes: The New Reactivity](https://stanza.dev/courses/svelte-5-runes) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*