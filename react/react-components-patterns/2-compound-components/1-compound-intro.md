---
source_course: "react-components-patterns"
source_lesson: "react-components-patterns-compound-intro"
---

# Compound Components: Working Together

Compound components are a pattern where multiple components work together to form a cohesive unit, sharing implicit state through Context.

## The Inspiration: HTML

Think of `<select>` and `<option>` in HTML:

```html
<select>
  <option value="a">Option A</option>
  <option value="b">Option B</option>
</select>
```

They work together - `<option>` is meaningless without `<select>`, and they share state implicitly.

## Building a Tabs Component

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// 1. Create Context
type TabsContextType = {
  activeTab: string;
  setActiveTab: (id: string) => void;
};

const TabsContext = createContext<TabsContextType | null>(null);

function useTabsContext() {
  const context = useContext(TabsContext);
  if (!context) {
    throw new Error('Tabs components must be used within <Tabs>');
  }
  return context;
}

// 2. Parent Component
type TabsProps = {
  defaultTab: string;
  children: ReactNode;
};

function Tabs({ defaultTab, children }: TabsProps) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// 3. Child Components
type TabListProps = { children: ReactNode };

function TabList({ children }: TabListProps) {
  return <div className="tab-list" role="tablist">{children}</div>;
}

type TabProps = {
  id: string;
  children: ReactNode;
};

function Tab({ id, children }: TabProps) {
  const { activeTab, setActiveTab } = useTabsContext();
  const isActive = activeTab === id;
  
  return (
    <button
      role="tab"
      aria-selected={isActive}
      className={`tab ${isActive ? 'active' : ''}`}
      onClick={() => setActiveTab(id)}
    >
      {children}
    </button>
  );
}

type TabPanelProps = {
  id: string;
  children: ReactNode;
};

function TabPanel({ id, children }: TabPanelProps) {
  const { activeTab } = useTabsContext();
  
  if (activeTab !== id) return null;
  
  return (
    <div role="tabpanel" className="tab-panel">
      {children}
    </div>
  );
}

// 4. Attach to parent for clean API
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

export { Tabs };
```

## Usage

```tsx
<Tabs defaultTab="profile">
  <Tabs.List>
    <Tabs.Tab id="profile">Profile</Tabs.Tab>
    <Tabs.Tab id="settings">Settings</Tabs.Tab>
    <Tabs.Tab id="notifications">Notifications</Tabs.Tab>
  </Tabs.List>
  
  <Tabs.Panel id="profile">
    <ProfileContent />
  </Tabs.Panel>
  <Tabs.Panel id="settings">
    <SettingsContent />
  </Tabs.Panel>
  <Tabs.Panel id="notifications">
    <NotificationsContent />
  </Tabs.Panel>
</Tabs>
```

## Benefits

1. **Flexible composition** - Arrange children however you want
2. **Implicit state** - No prop drilling
3. **Clean API** - Intuitive to use
4. **Separation of concerns** - Each component has one job

## When to Use Compound Components

✅ **Good for:**
- UI components with multiple related parts (Tabs, Accordion, Menu)
- When the relationship between parts is important
- When you want flexible composition

❌ **Overkill for:**
- Simple components with fixed structure
- Components that don't share state
- One-off implementations

## Resources

- [Compound Components Pattern](https://react.dev/learn/passing-data-deeply-with-context) — Using Context for implicit state sharing

---

> 📘 *This lesson is part of the [React Components & Patterns](https://stanza.dev/courses/react-components-patterns) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*