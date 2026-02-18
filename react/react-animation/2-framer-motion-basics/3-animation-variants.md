---
source_course: "react-animation"
source_lesson: "react-animation-variants"
---

# Variants for Complex Animations

Variants define animation states by name for cleaner code.

## Basic Variants

```jsx
const boxVariants = {
  hidden: { opacity: 0, scale: 0.8 },
  visible: { opacity: 1, scale: 1 },
};

function AnimatedBox() {
  return (
    <motion.div
      variants={boxVariants}
      initial="hidden"
      animate="visible"
    />
  );
}
```

## Variants with Transitions

```jsx
const buttonVariants = {
  idle: {
    scale: 1,
    boxShadow: '0 0 0 rgba(0, 0, 0, 0.1)',
  },
  hover: {
    scale: 1.05,
    boxShadow: '0 10px 20px rgba(0, 0, 0, 0.2)',
    transition: {
      type: 'spring',
      stiffness: 300,
    },
  },
  tap: {
    scale: 0.95,
  },
};

function Button({ children }) {
  return (
    <motion.button
      variants={buttonVariants}
      initial="idle"
      whileHover="hover"
      whileTap="tap"
    >
      {children}
    </motion.button>
  );
}
```

## Propagating Variants to Children

Variants automatically propagate to children:

```jsx
const containerVariants = {
  hidden: { opacity: 0 },
  visible: {
    opacity: 1,
    transition: {
      staggerChildren: 0.1, // Stagger child animations
    },
  },
};

const itemVariants = {
  hidden: { opacity: 0, y: 20 },
  visible: {
    opacity: 1,
    y: 0,
    transition: { duration: 0.3 },
  },
};

function StaggeredList({ items }) {
  return (
    <motion.ul
      variants={containerVariants}
      initial="hidden"
      animate="visible"
    >
      {items.map((item) => (
        <motion.li key={item.id} variants={itemVariants}>
          {item.name}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

## Dynamic Variants

```jsx
const itemVariants = {
  hidden: (index) => ({
    opacity: 0,
    x: index % 2 === 0 ? -50 : 50,
  }),
  visible: {
    opacity: 1,
    x: 0,
  },
};

function AlternatingList({ items }) {
  return (
    <motion.ul initial="hidden" animate="visible">
      {items.map((item, index) => (
        <motion.li
          key={item.id}
          custom={index}  // Pass custom value
          variants={itemVariants}
        >
          {item.name}
        </motion.li>
      ))}
    </motion.ul>
  );
}
```

## Orchestration

```jsx
const containerVariants = {
  hidden: {},
  visible: {
    transition: {
      staggerChildren: 0.1,      // Delay between children
      delayChildren: 0.3,        // Delay before first child
      staggerDirection: 1,       // 1 = forward, -1 = reverse
      when: 'beforeChildren',    // Parent animates first
    },
  },
};
```

## Menu Example

```jsx
const menuVariants = {
  closed: {
    clipPath: 'inset(10% 50% 90% 50% round 10px)',
    transition: {
      type: 'spring',
      bounce: 0,
      duration: 0.3,
    },
  },
  open: {
    clipPath: 'inset(0% 0% 0% 0% round 10px)',
    transition: {
      type: 'spring',
      bounce: 0,
      duration: 0.5,
      delayChildren: 0.2,
      staggerChildren: 0.05,
    },
  },
};

function Menu({ isOpen }) {
  return (
    <motion.nav
      variants={menuVariants}
      initial="closed"
      animate={isOpen ? 'open' : 'closed'}
    >
      {menuItems.map((item) => (
        <MenuItem key={item.id} {...item} />
      ))}
    </motion.nav>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*