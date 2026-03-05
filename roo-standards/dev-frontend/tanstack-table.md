## // TanStack Table v8 Configuration & Implementation Rules

name: tanstack_table_v8
description: Rules for implementing headless, type-safe tables using @tanstack/react-table.
globs: ["src/components/tables/**/*.{ts,tsx}", "src/hooks/use*Table.ts"]
tags:

- react
- table
- headless-ui
- typescript

---

# TanStack Table v8

## Core Concepts

- **Headless UI**: TanStack Table provides the **logic**, but you provide the **markup** (`<table>`, `<div>`, etc.).
- **TData Generic**: Define a type for your row data first. This ensures type safety across columns, cells, and filters.
- **The Table Instance**: Created via `useReactTable(options)`. It contains the state and APIs needed to render the UI.

---

## Required Setup: Data & Columns

Every table requires at least two options: `data` and `columns`.

### 1. Data Stability

In React, `data` and `columns` **must** have a stable reference to prevent infinite re-render loops.

- **Do**: Use `useMemo`, `useState`, or define them outside the component.
- **Don't**: Define them inline within the component body.

```tsx
// ✅ GOOD: Stable reference
const columns = useMemo<ColumnDef<User>[]>(
  () => [
    /*...*/
  ],
  [],
);
const [data] = useState<User[]>(initialData);

const table = useReactTable({
  data,
  columns,
  getCoreRowModel: getCoreRowModel(),
});
```

### 2. Column Definitions

Use `createColumnHelper<TData>()` for the best TypeScript experience.

| Column Type  | Method        | Purpose                                       | Data Model? |
| ------------ | ------------- | --------------------------------------------- | ----------- |
| **Accessor** | `.accessor()` | Extract/transform data for sorting/filtering. | **Yes**     |
| **Display**  | `.display()`  | Action buttons, checkboxes, custom UI.        | No          |
| **Grouping** | `.group()`    | Wrap other columns under a single header.     | No          |

#### Accessor Patterns:

- **Key**: `accessorKey: 'profile.firstName'` (supports dot notation).
- **Function**: `accessorFn: row => `${row.name} ${row.surname}``(requires a unique`id`).

---

## Row Models & Execution Order

TanStack Table is modular. You must import and provide the logic for features you want to use.

### The Row Model Pipeline

1. **`getCoreRowModel`**: (Required) Basic 1:1 mapping.
2. **`getFilteredRowModel`**: Applies column/global filters.
3. **`getGroupedRowModel`**: Handles aggregation and sub-rows.
4. **`getSortedRowModel`**: Reorders rows based on state.
5. **`getExpandedRowModel`**: Handles toggling sub-rows.
6. **`getPaginationRowModel`**: Slices rows for the current page.

> **Pro Tip**: Always use `table.getRowModel().rows` for the final rendering, as it accounts for all the transformations above.

---

## Rendering Guide

Since headers and cells can be strings, functions, or components, use the `flexRender` utility to safely output content.

```tsx
<thead>
  {table.getHeaderGroups().map((headerGroup) => (
    <tr key={headerGroup.id}>
      {headerGroup.headers.map((header) => (
        <th key={header.id} colSpan={header.colSpan}>
          {header.isPlaceholder
            ? null
            : flexRender(header.column.columnDef.header, header.getContext())}
        </th>
      ))}
    </tr>
  ))}
</thead>
```

---

## State Management

### 1. Controlled State (Hoisting)

To "lift" state (e.g., for server-side pagination), provide both `state` and the `on[State]Change` handler.

```tsx
const [sorting, setSorting] = useState<SortingState>([]);

const table = useReactTable({
  state: { sorting },
  onSortingChange: setSorting,
  // ...
});
```

### 2. Table State Strategies

- **`initialState`**: Set defaults (e.g., hidden columns) without managing the state yourself.
- **`state`**: Fully control the state. Note that if you provide `onSortingChange` but forget to pass the `sorting` value back into `state`, the UI will feel "frozen."

---

## Accessing Data

- **`row.original`**: The raw data object for that row.
- **`row.getValue(columnId)`**: The processed value (from the accessor).
- **`cell.getContext()`**: Provides everything needed for custom cell components (row, table, column, value).
