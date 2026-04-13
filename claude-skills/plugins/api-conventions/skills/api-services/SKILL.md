---
name: api-services
description: How to write service layer functions in the backend repository. Services contain business logic, Prisma transactions, custom exception handling, and SSE broadcasting.
---

# API Services

Services contain business logic, handle transactions, and throw custom exceptions. They sit between controllers and repositories.

## Standard Pattern

```javascript
import { PrismaClient } from "@prisma/client";
import { displayBus } from "../sse/displayBus.js";
import {
  BadRequestException,
  InternalServerException,
  NotFoundException,
} from "../../errors/index.js";
import { findById, createRecord, updateRecord } from "./ItemRepository.js";

const prisma = new PrismaClient();

// Simple read (no transaction needed)
export const getItemDetails = async (id) => {
  try {
    const item = await findById(id);
    if (!item) throw new NotFoundException("Item not found");
    return item;
  } catch (e) {
    if (e instanceof NotFoundException) throw e;
    console.log(e);
    throw new InternalServerException("Failed to get item");
  }
};

// Write with transaction
export const createItem = async (data) => {
  try {
    const result = await prisma.$transaction(async (tx) => {
      // Validate within transaction
      const existing = await findByName(data.name, tx);
      if (existing) throw new BadRequestException("Item already exists");

      // Create
      const item = await createRecord(data, tx);

      // Create related records in same transaction
      await createHistory({ itemId: item.id, type: "creation" }, tx);

      return item;
    });

    // Broadcast AFTER transaction commits (not inside)
    displayBus.broadcast(null);

    return result;
  } catch (e) {
    console.log(e);
    if (e instanceof NotFoundException || e instanceof BadRequestException) throw e;
    throw new InternalServerException("Failed to create item");
  }
};

// Update with transaction
export const updateItemFields = async (id, data, userId) => {
  try {
    return await prisma.$transaction(async (tx) => {
      const current = await findById(id, tx);
      if (!current) throw new NotFoundException("Item not found");

      const updated = await updateRecord({ id, data }, tx);

      // Track changes for history
      const changes = getChangedFields(current, data);
      if (Object.keys(changes).length) {
        await createHistory({
          itemId: id,
          userId,
          type: "field_update",
          value: changes,
        }, tx);
      }

      return updated;
    });
  } catch (e) {
    if (e instanceof NotFoundException) throw e;
    console.log(e);
    throw new InternalServerException("Failed to update item");
  }
};
```

## Rules

1. **Always wrap in try/catch**
2. **Re-throw** `NotFoundException` and `BadRequestException` (they carry meaningful user-facing messages)
3. **Wrap everything else** in `InternalServerException` with a generic message
4. **Use `prisma.$transaction(async (tx) => { ... })`** for multi-step write operations
5. **Pass `tx`** to ALL repository calls within the transaction
6. **Broadcast SSE** after transaction commits, not inside: `displayBus.broadcast(locationId)`
7. **No Prisma queries directly** in services — use repository functions
8. **Console.log errors** before re-throwing for server-side debugging
9. **Transaction timeout:** For heavy transactions, set explicit timeouts:
   ```javascript
   await prisma.$transaction(
     async (tx) => { ... },
     { maxWait: 10000, timeout: 15000 }
   );
   ```

## Service-to-Service Calls

Services can call other services for cross-module operations. Always pass the transaction:

```javascript
import { createAdminLaborTicket } from "../labor/LaborService.js";

export const handleShipment = async (payload) => {
  await prisma.$transaction(async (tx) => {
    // ... shipment logic
    await createAdminLaborTicket(laborPayload, tx);
  });
};
```

## Helper Functions in Services

Utility functions that don't need database access go in `[Module]Helper.js`:

```javascript
// In service
import { getAvailableQuantity, getNextRoute } from "./LaborHelper.js";

const { available } = getAvailableQuantity({ workOrder, routeId });
if (available < 0) throw new BadRequestException("Exceeds available quantity");
```
