---
name: frontend-drag-drop
description: DraggableWrapper kanban/list drag-and-drop system for the frontend repository. Covers single-list reorder, multi-column kanban boards, card components, and the onChange callback.
---

# Frontend Drag & Drop (DraggableWrapper)

## Overview

`DraggableWrapper` wraps `react-beautiful-dnd` and supports two modes:
- **Single list:** Reorder items within one droppable zone
- **Multi-column kanban:** Drag items between columns (stages)

Mode is auto-detected: if `elements[0].elements` is an array, it's multi-column.

## Multi-Column Kanban (Typical Usage)

```jsx
import { DraggableWrapper } from "../../components";

const KanbanBoard = ({ columns, onStageChange }) => {
  const handleChange = useCallback(
    (_newState, dragInfo) => {
      const { draggableId, toId } = dragInfo;
      onStageChange({
        id: Number(draggableId),
        stage: Number(toId),
      });
    },
    [onStageChange],
  );

  return (
    <DraggableWrapper
      elements={columns}
      Component={CardComponent}
      onChange={handleChange}
    />
  );
};
```

## Column Data Format

```javascript
// Each column = a droppable zone
const columns = [
  {
    id: "0",                    // droppable ID (string)
    label: "Prospecting",       // column header
    elements: [                 // cards in this column
      { id: "1", companyName: "Acme Corp", value: 50000 },
      { id: "2", companyName: "Widget Inc", value: 30000 },
    ],
  },
  {
    id: "1",
    label: "Qualifying",
    elements: [],
  },
  // ... more stages
];
```

## Card Component

The `Component` prop receives each draggable item's properties spread as props, plus `isDragging`:

```jsx
const LeadCard = ({ id, companyName, value, isDragging }) => (
  <Stack
    p="10px"
    sx={{
      backgroundColor: isDragging ? "hubColors.greyLighter" : "hubColors.white",
      borderRadius: "8px",
      boxShadow: isDragging ? 3 : 1,
    }}
  >
    <Typography variant="boldCaption">{companyName}</Typography>
    <Typography variant="caption">{getDollarValue(value)}</Typography>
  </Stack>
);
```

## onChange Callback

```javascript
onChange(newState, dragInfo)

// dragInfo contains:
{
  draggableId: "42",           // ID of the dragged item
  fromId: "0",                 // source column ID
  toId: "1",                   // destination column ID
  toIndex: 2,                  // position in destination
}
```

## DraggableWrapper Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `elements` | array | `[]` | Flat array (single list) or array of `{ id, elements }` (kanban) |
| `Component` | React.FC | required | Renders each draggable item |
| `onChange` | function | `() => {}` | Called on drag end: `(newState, dragInfo)` |
| `StartComponent` | React.FC | — | Rendered above each column's droppable area |
| `direction` | string | `"vertical"` | `"vertical"` or `"horizontal"` |
| `disablePlaceholder` | boolean | `false` | Hide drop target visual indicator |
| `includeSnapshot` | boolean | `false` | Pass drag snapshot to Component |
| `customDroppableProps` | object | `{}` | Extra props for Droppable |

## Single List Reorder

```jsx
<DraggableWrapper
  droppableId="my-list"
  elements={items}          // flat array: [{ id: "1", name: "A" }, ...]
  Component={ListItem}
  onChange={(reordered) => setItems(reordered)}
/>
```

## Integration with Mutations

Typically paired with an optimistic React Query mutation:

```jsx
const { mutate: updateStage } = usePutLeadStage();

<DraggableWrapper
  elements={columns}
  Component={LeadCard}
  onChange={(_newState, { draggableId, toId }) => {
    updateStage({ id: Number(draggableId), stage: Number(toId) });
  }}
/>
```

The mutation hook handles optimistic cache updates and rollback on error (see `frontend-react-query` skill).

## Rules

- Column `id` and element `id` must be **strings** (react-beautiful-dnd requirement)
- The `Component` receives all element properties spread as props
- `isDragging` boolean is always passed to Component
- Elements with `started: true` are drag-disabled (`isDragDisabled`)
- Visual placeholder shows a dashed blue border at the drop target
