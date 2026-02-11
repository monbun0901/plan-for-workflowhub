# DropdownFilter Component

**Version:** v1.0.0
**Status:** 🧩 Reusable Component
**Path:** `src/components/shared/DropdownFilter.tsx`

---

## 🎯 Purpose

Generic dropdown filter for single-select or multi-select filtering in ActionsBar.

---

## 📦 Props Interface

```typescript
interface FilterOption {
  value: string;
  label: string;
  icon?: ReactNode;
  count?: number;          // Badge count
}

interface DropdownFilterProps {
  label: string;                        // Filter button label
  options: FilterOption[];
  value?: string | string[];            // Single: string, Multi: string[]
  onChange: (value: string | string[]) => void;
  mode?: 'single' | 'multiple';        // Default: 'single'
  placeholder?: string;
  searchable?: boolean;                 // Enable search within options
}
```

---

## 🧬 Component Structure

```tsx
/**
 * @follows senior-architect: Controlled Dropdown Pattern
 * Headless UI Listbox hoặc Radix UI Select
 */
import { Listbox } from '@headlessui/react';
import { ChevronDownIcon, CheckIcon } from '@heroicons/react/24/outline';

export function DropdownFilter({
  label,
  options,
  value,
  onChange,
  mode = 'single',
  placeholder = 'Select...',
  searchable = false,
}: DropdownFilterProps) {
  const selectedCount = mode === 'multiple' && Array.isArray(value) ? value.length : 0;
  
  return (
    <Listbox value={value} onChange={onChange} multiple={mode === 'multiple'}>
      {({ open }) => (
        <div className="relative">
          <Listbox.Button className="flex items-center gap-2 rounded-md border border-gray-300 bg-white px-3 py-2 text-sm hover:bg-gray-50">
            <span className="font-medium text-gray-700">{label}</span>
            {selectedCount > 0 && (
              <span className="rounded-full bg-blue-100 px-2 py-0.5 text-xs text-blue-700">
                {selectedCount}
              </span>
            )}
            <ChevronDownIcon className={cn('h-4 w-4 transition-transform', open && 'rotate-180')} />
          </Listbox.Button>
          
          <Listbox.Options className="absolute z-20 mt-1 max-h-60 w-56 overflow-auto rounded-md border border-gray-200 bg-white py-1 shadow-lg">
            {options.map((option) => (
              <Listbox.Option
                key={option.value}
                value={option.value}
                className={({ active }) =>
                  cn(
                    'flex cursor-pointer items-center justify-between px-3 py-2 text-sm',
                    active && 'bg-blue-50'
                  )
                }
              >
                {({ selected }) => (
                  <>
                    <div className="flex items-center gap-2">
                      {option.icon}
                      <span className={selected ? 'font-semibold' : ''}>{option.label}</span>
                      {option.count !== undefined && (
                        <span className="text-xs text-gray-500">({option.count})</span>
                      )}
                    </div>
                    {selected && <CheckIcon className="h-4 w-4 text-blue-600" />}
                  </>
                )}
              </Listbox.Option>
            ))}
          </Listbox.Options>
        </div>
      )}
    </Listbox>
  );
}
```

---

## 📚 Usage Examples

### Single Select
```tsx
<DropdownFilter
  label="Status"
  mode="single"
  options={[
    { value: 'todo', label: 'To Do', count: 5 },
    { value: 'in_progress', label: 'In Progress', count: 3 },
    { value: 'done', label: 'Done', count: 12 },
  ]}
  value={statusFilter}
  onChange={setStatusFilter}
/>
```

### Multi-Select
```tsx
<DropdownFilter
  label="Assignees"
  mode="multiple"
  options={members.map(m => ({ value: m.id, label: m.name }))}
  value={selectedAssignees}
  onChange={setSelectedAssignees}
/>
```

### With Icons
```tsx
<DropdownFilter
  label="Priority"
  options={[
    { value: 'high', label: 'High', icon: <FlagIcon className="text-red-500" /> },
    { value: 'medium', label: 'Medium', icon: <FlagIcon className="text-yellow-500" /> },
    { value: 'low', label: 'Low', icon: <FlagIcon className="text-green-500" /> },
  ]}
  value={priorityFilter}
  onChange={setPriorityFilter}
/>
```

### Searchable Options
```tsx
<DropdownFilter
  label="Tags"
  mode="multiple"
  searchable
  options={allTags}
  value={selectedTags}
  onChange={setSelectedTags}
/>
```

---

## 🎨 Design Rules

- **Multi-Select Badge:** Show count badge khi có items selected.
- **Checkmark:** Hiển thị CheckIcon bên phải cho selected items.
- **Max Height:** Dropdown max `max-h-60` với scroll.
- **Z-Index:** `z-20` để dropdown không bị che bởi elements khác.

---

*Last Updated: 2026-02-11*
