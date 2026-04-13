---
name: frontend-tables
description: HubTableTemplate data table system for the frontend repository. Covers items/keys arrays, cell rendering (dates, currency, navigation, custom components), expandable rows, context menus, and the helper.js convention.
---

# Frontend Tables

## HubTableTemplate

Reusable table driven by `items` (header columns) and `keys` (cell renderers):

```jsx
import { HubTableTemplate } from "../../components";
import { items, keys } from "./helper";

const MyTable = ({ data, isLoading, meta }) => (
  <HubTableTemplate
    id="MyTable"
    headerProps={{ items, includeChevron: true }}
    bodyProps={{
      items: data,                    // row data array
      keys,                           // cell config array
      onRowClick: (e, item) => {},    // row click handler
      ContextMenu: MyContextMenu,     // right-click menu component
      RenderElement: ExpandableRow,   // expandable sub-row component
      RenderElementProps: { mode },   // props passed to RenderElement
      includeChevron: true,           // show expand/collapse arrow
    }}
    meta={meta}                       // { count, pages, limit, offset } for pagination
    isLoading={isLoading}
  />
);
```

## items Array (Header Columns)

```javascript
export const items = [
  { id: "workOrder", label: "Work Order" },
  { id: "quantity", label: "Qty" },
  { id: "dueDate", label: "Due Date" },
];
```

Each item defines a column header. The `id` is used for sorting.

## keys Array (Cell Renderers)

Each key maps a column to how its cell is rendered:

```javascript
export const keys = [
  // Navigate on click (blue link)
  {
    value: "workOrder",          // dot-notation path: "customer.name"
    variant: "blueBold",         // TableCell variant
    navigate: true,              // click navigates to route
    route: "works",              // navigate to /works/:id
    id: "workOrder",             // field for the route param
  },

  // Navigate with nested ID
  {
    value: "customer.name",
    variant: "blueBold",
    navigate: true,
    route: "customers",
    routeId: "customer.id",      // use routeId for nested paths
  },

  // Basic display with fallback
  { value: "quantity", defaultValue: 0 },

  // Auto-format as currency ($1,234.50)
  { value: "revenue", isCurrency: true },

  // Auto-format as date (Apr 13, 2026)
  { value: "dueDate", isDate: true },

  // Auto-format as date + time
  { value: "createdAt", isDateTime: true },

  // Checkbox display (disabled, read-only)
  { value: "isActive", isCheckbox: true },

  // User avatar chip
  { value: "assignee", isUser: true, userProps: { size: "small" } },

  // MUI Chip
  { value: "status", isChip: true, chipProps: { color: "primary" } },

  // Comma-separated array
  { value: "tags", isArray: true },

  // Custom component (receives { item } — the full row object)
  {
    value: "status",
    CustomComponent: StatusChip,
    sx: { width: "150px", maxWidth: "150px" },
  },

  // Click handler
  {
    value: "action",
    onClick: (item) => handleAction(item),
  },
];
```

## Available Key Properties

| Property | Type | Description |
|----------|------|-------------|
| `value` | string | Dot-notation path to data field |
| `defaultValue` | any | Fallback if value is null/undefined |
| `variant` | string | TableCell variant (`"blueBold"`) |
| `navigate` | boolean | Enable click navigation |
| `route` | string | URL prefix for navigation (`/route/:id`) |
| `id` / `routeId` | string | Field for navigation ID (dot-notation supported) |
| `isDate` | boolean | Format with `getDateData({ includeYear: true })` |
| `isDateTime` | boolean | Format with date + time |
| `isCurrency` | boolean | Format with `getDollarValue()` |
| `isCheckbox` | boolean | Render disabled Checkbox |
| `isUser` | boolean | Render UserChip component |
| `isChip` | boolean | Render MUI Chip |
| `isArray` | boolean | Render comma-separated values |
| `CustomComponent` | React.FC | Custom cell component receiving `{ item }` |
| `sx` | object | MUI sx prop for the TableCell |
| `onClick` | function | Click handler `(item) => {}` |
| `userProps` | object | Props forwarded to UserChip |
| `chipProps` | object | Props forwarded to Chip |

## Cell Processing Order

The `processValue()` function in `HubTableTemplate/helper.js` evaluates keys in this order:

1. `CustomComponent` (if valid React element)
2. `isUser` -> UserChip
3. `isChip` -> Chip
4. `isCheckbox` -> Checkbox
5. Skip null/falsy non-number values
6. `isCurrency` -> `getDollarValue()`
7. `isDate` -> `getDateData({ includeYear: true })`
8. `isArray` -> comma-joined spans
9. `isDateTime` -> `getDateData({ includeTime: true })`
10. Raw value

## Expandable Sub-Rows (RenderElement)

```jsx
// RenderElement receives { item, open, ...RenderElementProps }
const ExpandableRow = ({ item, open, mode }) => {
  const { data, isLoading } = useGetDetails(item.id);

  if (!open) return null;

  return (
    <Stack p="15px">
      <HubTableTemplate
        id={`SubTable-${item.id}`}
        headerProps={{ items: subItems }}
        bodyProps={{ items: data || [], keys: subKeys }}
        isLoading={isLoading}
      />
    </Stack>
  );
};

// In parent table
<HubTableTemplate
  bodyProps={{
    items: data,
    keys,
    includeChevron: true,        // show expand arrow
    RenderElement: ExpandableRow,
    RenderElementProps: { mode },
  }}
/>
```

Clicking a row toggles expand/collapse. Only one row is open at a time.

## CustomComponent Pattern

When you need custom cell rendering beyond the built-in formatters:

```jsx
// In helper.js (if small) or its own file (if >30 lines)
const StatusChip = ({ item }) => {
  const color = STATUS_COLORS[item.health] || "green";
  return (
    <Stack direction="row" gap="4px" alignItems="center">
      <span style={{ width: 6, height: 6, borderRadius: "50%", backgroundColor: color }} />
      <Typography variant="caption" fontSize={10}>{item.statusLabel}</Typography>
    </Stack>
  );
};

// In keys array
{ value: "health", CustomComponent: StatusChip }
```

`CustomComponent` receives `{ item }` — the **full row object**, not just the cell value. This lets you access any field for rendering.

## Table Folder Convention

```
TableName/
  TableName.jsx       # Fetches data, renders HubTableTemplate
  helper.js           # items[] and keys[] arrays, small CustomComponents
  styles.js           # Styled components (StyledTableRow, etc.)
  index.js            # Barrel export
```

The `.jsx` file should be minimal — just data fetching and `<HubTableTemplate>`. All column/cell config goes in `helper.js`. If a `CustomComponent` grows beyond ~30 lines, extract it to its own `.jsx` file in the same folder.
