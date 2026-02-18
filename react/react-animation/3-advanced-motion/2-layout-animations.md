---
source_course: "react-animation"
source_lesson: "react-animation-layout-animations"
---

# Layout Animations

Automatically animate layout changes.

## The layout Prop

```jsx
function ExpandableCard() {
  const [isExpanded, setIsExpanded] = useState(false);

  return (
    <motion.div
      layout  // Enable layout animations
      onClick={() => setIsExpanded(!isExpanded)}
      style={{
        width: isExpanded ? 300 : 150,
        height: isExpanded ? 200 : 100,
      }}
      className="card"
    />
  );
}
```

## Layout Animation Types

```jsx
// Animate position only
<motion.div layout="position" />

// Animate size only
<motion.div layout="size" />

// Animate both (default)
<motion.div layout />
```

## Reordering Lists

```jsx
import { Reorder } from 'framer-motion';

function ReorderableList() {
  const [items, setItems] = useState([1, 2, 3, 4, 5]);

  return (
    <Reorder.Group values={items} onReorder={setItems}>
      {items.map((item) => (
        <Reorder.Item key={item} value={item}>
          Item {item}
        </Reorder.Item>
      ))}
    </Reorder.Group>
  );
}
```

## Layout Groups

Coordinate animations across sibling components:

```jsx
import { LayoutGroup } from 'framer-motion';

function Tabs({ tabs, activeTab, setActiveTab }) {
  return (
    <LayoutGroup>
      <div className="tabs">
        {tabs.map((tab) => (
          <button
            key={tab.id}
            onClick={() => setActiveTab(tab.id)}
            className="tab"
          >
            {tab.label}
            {activeTab === tab.id && (
              <motion.div
                layoutId="activeTab"  // Same layoutId = shared animation
                className="active-indicator"
              />
            )}
          </button>
        ))}
      </div>
    </LayoutGroup>
  );
}
```

## Shared Layout Animation

```jsx
function Gallery({ items }) {
  const [selectedId, setSelectedId] = useState(null);

  return (
    <>
      <div className="grid">
        {items.map((item) => (
          <motion.div
            key={item.id}
            layoutId={`card-${item.id}`}
            onClick={() => setSelectedId(item.id)}
            className="card"
          >
            <motion.h2 layoutId={`title-${item.id}`}>
              {item.title}
            </motion.h2>
          </motion.div>
        ))}
      </div>

      <AnimatePresence>
        {selectedId && (
          <motion.div
            layoutId={`card-${selectedId}`}
            className="card-expanded"
            onClick={() => setSelectedId(null)}
          >
            <motion.h2 layoutId={`title-${selectedId}`}>
              {items.find((i) => i.id === selectedId).title}
            </motion.h2>
            <p>Full description here...</p>
          </motion.div>
        )}
      </AnimatePresence>
    </>
  );
}
```

## Transition Configuration

```jsx
<motion.div
  layout
  transition={{
    layout: {
      type: 'spring',
      stiffness: 500,
      damping: 30,
    },
  }}
/>
```

---

> 📘 *This lesson is part of the [React Animation & Motion](https://stanza.dev/courses/react-animation) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*