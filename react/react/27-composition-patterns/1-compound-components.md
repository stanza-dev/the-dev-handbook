---
source_course: "react"
source_lesson: "react-compound-components"
---

# Compound Components Pattern

Compound components are a set of components that work together to form a complete UI pattern.

## Benefits

- **Flexible API**: Users can arrange children as needed
- **Shared state**: Components implicitly share state
- **Cleaner markup**: Semantic, readable JSX

## Examples

- `<Select>` with `<Option>`
- `<Tabs>` with `<Tab>` and `<TabPanel>`
- `<Accordion>` with `<AccordionItem>`

## Code Examples

**Compound Components pattern with Tabs**

```tsx
import { createContext, useContext, useState, ReactNode } from 'react';

// Context for sharing state
type TabsContextType = {
  activeTab: string;
  setActiveTab: (id: string) => void;
};

const TabsContext = createContext<TabsContextType | null>(null);

function useTabs() {
  const context = useContext(TabsContext);
  if (!context) throw new Error('Must be used within Tabs');
  return context;
}

// Main Tabs component
function Tabs({ children, defaultTab }: { children: ReactNode; defaultTab: string }) {
  const [activeTab, setActiveTab] = useState(defaultTab);
  
  return (
    <TabsContext.Provider value={{ activeTab, setActiveTab }}>
      <div className="tabs">{children}</div>
    </TabsContext.Provider>
  );
}

// Tab list container
function TabList({ children }: { children: ReactNode }) {
  return <div className="tab-list" role="tablist">{children}</div>;
}

// Individual tab button
function Tab({ id, children }: { id: string; children: ReactNode }) {
  const { activeTab, setActiveTab } = useTabs();
  
  return (
    <button
      role="tab"
      aria-selected={activeTab === id}
      onClick={() => setActiveTab(id)}
      className={activeTab === id ? 'active' : ''}
    >
      {children}
    </button>
  );
}

// Tab panel content
function TabPanel({ id, children }: { id: string; children: ReactNode }) {
  const { activeTab } = useTabs();
  if (activeTab !== id) return null;
  
  return <div role="tabpanel">{children}</div>;
}

// Attach sub-components
Tabs.List = TabList;
Tabs.Tab = Tab;
Tabs.Panel = TabPanel;

// Usage - clean, flexible API
function App() {
  return (
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
  );
}
```


## Resources

- [React Patterns](https://www.patterns.dev/react) — Comprehensive React patterns guide

---

> 📘 *This lesson is part of the [React Mastery](https://stanza.dev/courses/react) course on [Stanza](https://stanza.dev) — the IDE-native learning platform for developers.*