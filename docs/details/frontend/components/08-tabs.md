# Tabs Component

**Version:** v1.0.0
**Status:** 🧩 Reusable Component
**Path:** `src/components/shared/Tabs.tsx`

---

## 🎯 Purpose

Generic tabs navigation pattern for multi-section pages (e.g., Project Detail tabs: Overview, Tasks, Issues).

---

## 📦 Props Interface

```typescript
interface TabItem {
  key: string;
  label: string;
  icon?: ReactNode;
  count?: number;          // Badge count (e.g., Issues: 3)
  disabled?: boolean;
}

interface TabsProps {
  items: TabItem[];
  activeKey: string;
  onChange: (key: string) => void;
  variant?: 'underline' | 'pills';  // Style variant
}
```

---

## 🧬 Component Structure

```tsx
/**
 * @follows senior-architect: Controlled Component Pattern
 * Tabs không quản lý state, parent component control activeKey
 */
export function Tabs({ items, activeKey, onChange, variant = 'underline' }: TabsProps) {
  return (
    <nav className={cn('flex gap-1', variant === 'underline' && 'border-b')}>
      {items.map((item) => (
        <button
          key={item.key}
          onClick={() => !item.disabled && onChange(item.key)}
          disabled={item.disabled}
          className={cn(
            'flex items-center gap-2 px-4 py-2 text-sm font-medium transition-colors',
            variant === 'underline' && activeKey === item.key
              ? 'border-b-2 border-blue-600 text-blue-600'
              : 'text-gray-600 hover:text-gray-900',
            variant === 'pills' && activeKey === item.key
              ? 'bg-blue-100 text-blue-700 rounded-md'
              : 'hover:bg-gray-100 rounded-md',
            item.disabled && 'opacity-50 cursor-not-allowed'
          )}
        >
          {item.icon}
          <span>{item.label}</span>
          {item.count !== undefined && (
            <span className="ml-1 rounded-full bg-gray-200 px-2 py-0.5 text-xs">
              {item.count}
            </span>
          )}
        </button>
      ))}
    </nav>
  );
}
```

---

## 📚 Usage Examples

### Basic Tabs (Underline)
```tsx
const [activeTab, setActiveTab] = useState('overview');

<Tabs
  items={[
    { key: 'overview', label: 'Overview' },
    { key: 'tasks', label: 'Tasks', count: 12 },
    { key: 'issues', label: 'Issues', count: 3 },
  ]}
  activeKey={activeTab}
  onChange={setActiveTab}
/>

{activeTab === 'overview' && <OverviewContent />}
{activeTab === 'tasks' && <TasksContent />}
```

### Pills Variant with Icons
```tsx
<Tabs
  variant="pills"
  items={[
    { key: 'profile', label: 'Profile', icon: <UserIcon /> },
    { key: 'organization', label: 'Organization', icon: <BuildingIcon /> },
    { key: 'app', label: 'App Settings', icon: <CogIcon /> },
  ]}
  activeKey={activeTab}
  onChange={setActiveTab}
/>
```

### With Disabled Tab
```tsx
<Tabs
  items={[
    { key: 'active', label: 'Active' },
    { key: 'pending', label: 'Pending', disabled: true },
  ]}
  activeKey="active"
  onChange={setActiveTab}
/>
```

---

## 🎨 Design Rules

- **Controlled:** Parent component owns active state (React best practice).
- **URL Sync:** Recommended: sync activeKey với URL query params.
- **Mobile:** Tabs scroll horizontally trên mobile.
- **A11y:** Use `role="tablist"` and proper ARIA attributes.

---

*Last Updated: 2026-02-11*
