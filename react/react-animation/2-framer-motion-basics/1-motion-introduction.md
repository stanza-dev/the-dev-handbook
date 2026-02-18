---
source_course: "react-animation"
source_lesson: "react-animation-motion-introduction"
---

# Introduction to Framer Motion

Framer Motion is a production-ready React animation library.

## Installation

```bash
npm install framer-motion
```

## Basic Animation

```jsx
import { motion } from 'framer-motion';

function FadeIn() {
  return (
    <motion.div
      initial={{ opacity: 0 }}
      animate={{ opacity: 1 }}
      transition={{ duration: 0.5 }}
    >
      Hello, I fade in!
    </motion.div>
  );
}
```

## Motion Components

Replace HTML elements with `motion.` versions:

```jsx
// Instead of <div>, use <motion.div>
<motion.div />
<motion.span />
<motion.button />
<motion.ul />
<motion.svg />
// etc.
```

## Animation Props

```jsx
<motion.div
  initial={{ opacity: 0, y: 20 }}    // Starting state
  animate={{ opacity: 1, y: 0 }}     // Target state
  exit={{ opacity: 0, y: -20 }}      // Exit state (with AnimatePresence)
  transition={{ duration: 0.3 }}     // How to animate
/>
```

## Animatable Properties

```jsx
<motion.div
  animate={{
    // Transform
    x: 100,               // translateX
    y: 50,                // translateY
    rotate: 90,           // rotation in degrees
    scale: 1.2,           // scale
    rotateX: 45,          // 3D rotation
    rotateY: 45,
    
    // Style
    opacity: 0.5,
    backgroundColor: '#ff0000',
    color: '#ffffff',
    borderRadius: '50%',
    boxShadow: '0 10px 20px rgba(0,0,0,0.2)',
  }}
/>
```

## State-Based Animation

```jsx
function ToggleCard() {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <motion.div
      animate={{
        height: isExpanded ? 200 : 100,
        backgroundColor: isExpanded ? '#3b82f6' : '#e5e7eb'
      }}
      transition={{ duration: 0.3 }}
      onClick={() => setIsExpanded(!isExpanded)}
    >
      Click to toggle
    </motion.div>
  );
}
```

## Hover and Tap Animations

```jsx
function InteractiveButton() {
  return (
    <motion.button
      whileHover={{ scale: 1.05 }}
      whileTap={{ scale: 0.95 }}
      transition={{ type: 'spring', stiffness: 400, damping: 17 }}
    >
      Click me
    </motion.button>
  );
}
```

## Spring Physics

```jsx
<motion.div
  animate={{ x: 100 }}
  transition={{
    type: 'spring',
    stiffness: 100,    // Spring stiffness
    damping: 10,       // Friction/resistance
    mass: 1,           // Object mass
    bounce: 0.5,       // Bounciness (0-1)
  }}
/>
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*