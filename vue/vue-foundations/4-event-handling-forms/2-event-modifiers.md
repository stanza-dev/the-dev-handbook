---
source_course: "vue-foundations"
source_lesson: "vue-foundations-event-modifiers"
---

# Event Modifiers

Event modifiers are special suffixes that handle common event manipulation tasks directly in the template. They replace the need to call `event.preventDefault()` or `event.stopPropagation()` in your methods.

## Common Event Modifiers

### .prevent - Prevent Default Behavior

```vue
<template>
  <!-- Without modifier -->
  <form @submit="handleSubmit">
    <!-- Must call event.preventDefault() in handler -->
  </form>
  
  <!-- With modifier - cleaner! -->
  <form @submit.prevent="handleSubmit">
    <!-- Default form submission is prevented -->
  </form>
  
  <!-- Prevent default without a handler -->
  <form @submit.prevent>
    <!-- Just prevents the default, no handler needed -->
  </form>
  
  <!-- Prevent link navigation -->
  <a href="/" @click.prevent="handleClick">Stay on page</a>
</template>
```

### .stop - Stop Event Propagation

```vue
<script setup>
function handleParentClick() {
  console.log('Parent clicked')
}

function handleChildClick() {
  console.log('Child clicked')
}
</script>

<template>
  <div @click="handleParentClick" class="parent">
    <!-- Without .stop, clicking button logs both messages -->
    <button @click="handleChildClick">Normal</button>
    
    <!-- With .stop, only child handler runs -->
    <button @click.stop="handleChildClick">Stopped</button>
  </div>
</template>
```

### .once - Only Trigger Once

```vue
<template>
  <!-- Handler runs only on first click -->
  <button @click.once="showWelcome">Show Welcome (once)</button>
  
  <!-- Useful for one-time initialization -->
  <video @play.once="trackVideoStart" />
</template>
```

### .self - Only Trigger on Element Itself

```vue
<template>
  <!-- Only fires when clicking the div itself, not children -->
  <div @click.self="handleDivClick" class="container">
    <button>Clicking me won't trigger div handler</button>
    <p>Clicking me won't trigger div handler either</p>
  </div>
</template>
```

### .capture - Use Capture Mode

```vue
<template>
  <!-- Handle event during capture phase (parent first) -->
  <div @click.capture="handleCapture">
    <button @click="handleClick">Click</button>
  </div>
  <!-- handleCapture runs BEFORE handleClick -->
</template>
```

### .passive - Improve Scroll Performance

```vue
<template>
  <!-- Don't block scrolling while handler runs -->
  <div @scroll.passive="onScroll">
    Long scrollable content...
  </div>
  
  <!-- Especially useful for touch events -->
  <div @touchmove.passive="onTouchMove">...</div>
</template>
```

## Chaining Modifiers

Modifiers can be chained together:

```vue
<template>
  <!-- Stop propagation AND prevent default -->
  <a @click.stop.prevent="handleClick" href="/">Link</a>
  
  <!-- Only trigger once AND prevent default -->
  <form @submit.once.prevent="handleSubmit">...</form>
  
  <!-- Order matters for .self -->
  <div @click.self.prevent="onClick">...</div>
</template>
```

## Key Modifiers

Listen for specific keyboard keys:

```vue
<template>
  <!-- Common keys -->
  <input @keyup.enter="submit" />
  <input @keyup.tab="nextField" />
  <input @keyup.delete="clearInput" />  <!-- Both Delete and Backspace -->
  <input @keyup.esc="cancel" />
  <input @keyup.space="togglePlay" />
  
  <!-- Arrow keys -->
  <input @keyup.up="previousItem" />
  <input @keyup.down="nextItem" />
  <input @keyup.left="goBack" />
  <input @keyup.right="goForward" />
</template>
```

### System Modifier Keys

```vue
<template>
  <!-- Ctrl/Cmd + key combinations -->
  <input @keyup.ctrl.enter="submitForm" />
  <input @keyup.meta.s="saveDocument" />  <!-- Cmd on Mac -->
  <input @keyup.alt.a="selectAll" />
  <input @keyup.shift.tab="previousField" />
  
  <!-- Multiple modifiers -->
  <input @keyup.ctrl.shift.s="saveAs" />
</template>
```

### .exact Modifier

Require exact modifier combination:

```vue
<template>
  <!-- Fires even if Alt or Shift is also pressed -->
  <button @click.ctrl="onClick">A</button>
  
  <!-- Only fires when ONLY Ctrl is pressed -->
  <button @click.ctrl.exact="onCtrlClick">A</button>
  
  <!-- Only fires when NO modifiers are pressed -->
  <button @click.exact="onClick">A</button>
</template>
```

## Mouse Button Modifiers

```vue
<template>
  <button @click.left="onLeftClick">Left Click</button>
  <button @click.right="onRightClick">Right Click</button>
  <button @click.middle="onMiddleClick">Middle Click</button>
  
  <!-- Prevent context menu on right click -->
  <div @contextmenu.prevent="showCustomMenu">
    Right-click for custom menu
  </div>
</template>
```

## Practical Example: Keyboard Navigation

```vue
<script setup>
import { ref } from 'vue'

const items = ref(['Home', 'About', 'Contact', 'Help'])
const selectedIndex = ref(0)

function selectPrevious() {
  if (selectedIndex.value > 0) {
    selectedIndex.value--
  }
}

function selectNext() {
  if (selectedIndex.value < items.value.length - 1) {
    selectedIndex.value++
  }
}

function activateItem() {
  console.log('Activated:', items.value[selectedIndex.value])
}
</script>

<template>
  <ul 
    tabindex="0"
    @keyup.up.prevent="selectPrevious"
    @keyup.down.prevent="selectNext"
    @keyup.enter="activateItem"
  >
    <li 
      v-for="(item, index) in items" 
      :key="item"
      :class="{ selected: index === selectedIndex }"
      @click="selectedIndex = index"
    >
      {{ item }}
    </li>
  </ul>
</template>
```

## Resources

- [Event Modifiers](https://vuejs.org/guide/essentials/event-handling.html#event-modifiers) — Official documentation on event modifiers

---

> 📘 *This lesson is part of the [Vue Foundations](https://stanza.dev/courses/vue-foundations) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*