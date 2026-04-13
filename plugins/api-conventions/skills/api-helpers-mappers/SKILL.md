---
name: api-helpers-mappers
description: How to write helper and mapper functions in the backend repository. Helpers contain pure utility functions (calculations, grouping, dates). Mappers transform raw DB records into API response shapes.
---

# API Helpers & Mappers

## Helpers

Helpers contain **pure utility functions** with no database access. They handle calculations, grouping, date math, and business logic:

```javascript
// LaborHelper.js
import { addDays, isWeekend } from "date-fns";

export const addBusinessDays = (date, days) => {
  let result = new Date(date);
  let addedDays = 0;
  const increments = Math.abs(Math.floor(days / 1440));
  const i = days > 0 ? 1 : -1;

  while (addedDays < increments) {
    result = addDays(result, i);
    if (!isWeekend(result)) addedDays++;
  }

  return new Date(result).toISOString();
};

export const getAvailableQuantity = ({ workOrder, routeId }) => {
  const curr = workOrder?.routing?.findIndex((r) => r?.id === routeId);
  const lastRoute = curr > 0 && workOrder?.routing[curr - 1];
  const routing = workOrder.routing[curr];

  const completed = routing?.completed || 0;
  const scrapped = routing?.scrapped || 0;
  const base = lastRoute?.completed || workOrder.quantity;
  const available = base - completed - scrapped;

  return { available, availableScrap: base - scrapped, base };
};

export const getNextRoute = ({ workOrder, routeId }) => {
  const parentRoutes = workOrder.routing.filter(
    (r) => !r.parentRouteId && (r.waxInstructions || r.assemblyInstructions || /* ... */)
  );
  const curr = parentRoutes.findIndex((r) => r?.id === routeId);
  if (curr < parentRoutes.length - 1) return parentRoutes[curr + 1].id;
  return false;
};
```

### Grouping Helper

```javascript
// SalesHelper.js
import { LEAD_STAGE_LABELS, QUOTE_STAGE_LABELS } from "../../constants/salesStages.js";

export const groupByStage = (items, stageLabels) =>
  stageLabels.map(({ id, label }) => ({
    id: String(id),
    label,
    elements: items
      .filter((item) => item.stage === id)
      .map((item) => ({ ...item, id: String(item.id) })),
  }));

export const groupLeads = (leads) => groupByStage(leads, LEAD_STAGE_LABELS);
export const groupQuotes = (quotes) => groupByStage(quotes, QUOTE_STAGE_LABELS);
```

### Date Recalculation Helper

```javascript
export const updatePlannedDates = (routes, startDate) => {
  if (routes.length === 0) return routes;

  let plannedStartDate = startDate;
  return routes.map((r) => {
    const plannedEndDate = addBusinessDays(plannedStartDate, r?.leadTime || 0);
    const updatedRoute = { ...r, plannedStartDate, plannedEndDate };
    plannedStartDate = plannedEndDate;
    return updatedRoute;
  });
};
```

## Mappers

Mappers transform **raw Prisma DB records** into **API response shapes**. They flatten nested relations and compute derived fields:

```javascript
// LaborMapper.js
export const TicketMapper = (tickets = []) =>
  tickets.map((t) => ({
    id: t.id,
    workOrder: t.routing.workOrder.workOrder,
    userId: t.user.id,
    userName: t.user.name,
    part: t.routing.workOrder.part,
    completed: t.completed,
    scrapped: t.scrapped,
  }));

export const mapLaborData = (labor) => {
  const routing = labor?.routing;
  const workOrder = routing?.workOrder;

  const completed = routing?.completed || 0;
  const scrapped = routing?.scrapped || 0;
  const base = routing?.quantity || 0;
  const available = base - completed - scrapped;

  return {
    id: labor?.id,
    completed: labor?.completed || 0,
    scrapped: labor?.scrapped || 0,
    startDate: labor?.startDate,
    routing: { id: routing?.id, quantity: base, completed, scrapped },
    workOrder: {
      workOrder: workOrder?.workOrder,
      part: workOrder?.part,
    },
    available,  // computed field
  };
};
```

## Rules

- **Helpers:** Pure functions, no database calls, no side effects. Import constants if needed.
- **Mappers:** Transform DB shape to API shape. Flatten nested relations. Compute derived fields (available qty, totals, etc.).
- Both live in `src/[module]/[Module]Helper.js` and `[Module]Mapper.js`.
- Services call helpers/mappers but never the reverse.
- Keep functions small and single-purpose.
