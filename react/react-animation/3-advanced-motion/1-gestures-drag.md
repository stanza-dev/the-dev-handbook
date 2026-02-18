---
source_course: "react-animation"
source_lesson: "react-animation-gestures-drag"
---

# Gesture Animations

Framer Motion provides powerful gesture recognition.

## Hover, Tap, Focus

```jsx
<motion.button
  whileHover={{ scale: 1.1, backgroundColor: '#3b82f6' }}
  whileTap={{ scale: 0.9 }}
  whileFocus={{ boxShadow: '0 0 0 3px rgba(59, 130, 246, 0.5)' }}
>
  Interactive Button
</motion.button>
```

## Drag

```jsx
function DraggableCard() {
  return (
    <motion.div
      drag                          // Enable drag
      dragConstraints={{
        top: -100,
        right: 100,
        bottom: 100,
        left: -100,
      }}
      dragElastic={0.2}              // Elasticity at constraints
      dragMomentum={true}            // Continue with momentum
      whileDrag={{ scale: 1.1 }}
      className="card"
    >
      Drag me!
    </motion.div>
  );
}
```

## Drag with Ref Constraints

```jsx
function DragWithinContainer() {
  const constraintsRef = useRef(null);

  return (
    <motion.div ref={constraintsRef} className="container">
      <motion.div
        drag
        dragConstraints={constraintsRef}
        className="draggable"
      />
    </motion.div>
  );
}
```

## Drag Direction

```jsx
// Only horizontal drag
<motion.div drag="x" />

// Only vertical drag
<motion.div drag="y" />
```

## Drag Events

```jsx
function DragEvents() {
  return (
    <motion.div
      drag
      onDragStart={(event, info) => {
        console.log('Started at:', info.point);
      }}
      onDrag={(event, info) => {
        console.log('Current offset:', info.offset);
        console.log('Current velocity:', info.velocity);
      }}
      onDragEnd={(event, info) => {
        console.log('Ended with velocity:', info.velocity);
      }}
    />
  );
}
```

## Swipe to Delete

```jsx
function SwipeableItem({ onDelete, children }) {
  return (
    <motion.div
      drag="x"
      dragConstraints={{ left: 0, right: 0 }}
      onDragEnd={(_, info) => {
        if (info.offset.x < -100) {
          onDelete();
        }
      }}
      className="swipeable-item"
    >
      {children}
    </motion.div>
  );
}
```

## Pan Gesture

```jsx
function PanHandler() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  return (
    <motion.div
      onPan={(event, info) => {
        setPosition({
          x: position.x + info.delta.x,
          y: position.y + info.delta.y,
        });
      }}
      animate={position}
      className="pan-target"
    />
  );
}
```

## useMotionValue for Performance

```jsx
import { motion, useMotionValue, useTransform } from 'framer-motion';

function RotatingCard() {
  const x = useMotionValue(0);
  const rotate = useTransform(x, [-200, 200], [-30, 30]);
  const background = useTransform(
    x,
    [-200, 0, 200],
    ['#ff008c', '#7700ff', '#00d4ff']
  );

  return (
    <motion.div
      style={{ x, rotate, background }}
      drag="x"
      dragConstraints={{ left: 0, right: 0 }}
    />
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*