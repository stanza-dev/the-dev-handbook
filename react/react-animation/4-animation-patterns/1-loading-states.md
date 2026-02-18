---
source_course: "react-animation"
source_lesson: "react-animation-loading-states"
---

# Loading State Animations

Engaging loading indicators improve perceived performance.

## Skeleton Loading

```jsx
function Skeleton({ width, height }) {
  return (
    <motion.div
      className="skeleton"
      style={{ width, height }}
      animate={{
        background: [
          'linear-gradient(90deg, #e5e7eb 0%, #f3f4f6 50%, #e5e7eb 100%)',
          'linear-gradient(90deg, #f3f4f6 0%, #e5e7eb 50%, #f3f4f6 100%)',
        ],
        backgroundSize: '200% 100%',
        backgroundPosition: ['100% 0', '-100% 0'],
      }}
      transition={{
        duration: 1.5,
        repeat: Infinity,
        ease: 'linear',
      }}
    />
  );
}

function CardSkeleton() {
  return (
    <div className="card">
      <Skeleton width="100%" height={200} />
      <Skeleton width="60%" height={24} />
      <Skeleton width="80%" height={16} />
    </div>
  );
}
```

## Pulsing Dots

```jsx
function PulsingDots() {
  return (
    <div className="dots-container">
      {[0, 1, 2].map((index) => (
        <motion.div
          key={index}
          className="dot"
          animate={{
            scale: [1, 1.5, 1],
            opacity: [0.5, 1, 0.5],
          }}
          transition={{
            duration: 1,
            repeat: Infinity,
            delay: index * 0.2,
          }}
        />
      ))}
    </div>
  );
}
```

## Spinning Loader

```jsx
function SpinningLoader() {
  return (
    <motion.div
      className="spinner"
      animate={{ rotate: 360 }}
      transition={{
        duration: 1,
        repeat: Infinity,
        ease: 'linear',
      }}
    />
  );
}
```

## Progress Bar

```jsx
function ProgressBar({ progress }) {
  return (
    <div className="progress-track">
      <motion.div
        className="progress-fill"
        initial={{ width: 0 }}
        animate={{ width: `${progress}%` }}
        transition={{ duration: 0.3 }}
      />
    </div>
  );
}
```

## Content Loading Transition

```jsx
function DataDisplay({ data, isLoading }) {
  return (
    <AnimatePresence mode="wait">
      {isLoading ? (
        <motion.div
          key="loading"
          initial={{ opacity: 0 }}
          animate={{ opacity: 1 }}
          exit={{ opacity: 0 }}
        >
          <CardSkeleton />
        </motion.div>
      ) : (
        <motion.div
          key="content"
          initial={{ opacity: 0, y: 10 }}
          animate={{ opacity: 1, y: 0 }}
          exit={{ opacity: 0, y: -10 }}
        >
          <Card data={data} />
        </motion.div>
      )}
    </AnimatePresence>
  );
}
```

## Button Loading State

```jsx
function LoadingButton({ isLoading, children, ...props }) {
  return (
    <motion.button
      {...props}
      disabled={isLoading}
      whileTap={!isLoading ? { scale: 0.95 } : {}}
    >
      <AnimatePresence mode="wait">
        {isLoading ? (
          <motion.span
            key="loading"
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
          >
            <SpinningLoader />
          </motion.span>
        ) : (
          <motion.span
            key="content"
            initial={{ opacity: 0 }}
            animate={{ opacity: 1 }}
            exit={{ opacity: 0 }}
          >
            {children}
          </motion.span>
        )}
      </AnimatePresence>
    </motion.button>
  );
}
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*