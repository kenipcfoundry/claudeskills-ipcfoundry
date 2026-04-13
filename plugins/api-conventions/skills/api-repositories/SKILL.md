---
name: api-repositories
description: How to write Prisma data access repositories in the backend repository. Repositories handle all database queries with optional transaction parameters, select-based field selection, and the Module resolution pattern.
---

# API Repositories

Repositories handle all Prisma queries. Every function accepts an **optional transaction parameter** so it works both standalone and within a `prisma.$transaction()`.

## Module Resolution Pattern

```javascript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();
const Items = prisma.item;
const ItemHistory = prisma.itemHistory;

// Pattern: transaction?.modelName ?? DefaultModel
export const findById = async (id, transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.findUnique({
    where: { id },
    select: {
      id: true,
      name: true,
      createdAt: true,
    },
  });
};
```

**Every repository function follows this structure:**
1. Accept `transaction = null` as the last parameter
2. First line resolves the module: `const Module = transaction?.modelName ?? DefaultModel`
3. Use `Module` for all Prisma calls
4. Return the Prisma result directly (no wrapping)

## CRUD Examples

### Find Many (with pagination)

```javascript
export const findAll = async ({ locationId, pagination }, transaction = null) => {
  const Module = transaction?.item ?? Items;
  const { orderBy, sortOrder, limit, offset } = pagination;

  const where = { locationId };

  const items = await Module.findMany({
    where,
    select: {
      id: true,
      name: true,
      status: true,
      customer: { select: { id: true, name: true } },
    },
    orderBy: { [orderBy]: sortOrder },
    take: limit,
    skip: offset,
  });

  const count = await Module.count({ where });
  return { items, count };
};
```

### Find Unique

```javascript
export const findById = async (id, transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.findUnique({
    where: { id },
    select: {
      id: true,
      name: true,
      quantity: true,
      customer: { select: { id: true, name: true } },
      history: {
        orderBy: { createdAt: "desc" },
        select: { id: true, type: true, value: true, createdAt: true },
      },
    },
  });
};
```

### Find First

```javascript
export const findLatestByPrefix = async (prefix, transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.findFirst({
    where: { number: { startsWith: prefix } },
    orderBy: { number: "desc" },
    select: { number: true },
  });
};
```

### Create

```javascript
export const createRecord = async (data, transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.create({ data });
};
```

### Create Many

```javascript
export const createBatch = async (data = [], transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.createMany({ data });
};
```

### Update

```javascript
export const updateRecord = async ({ id, data }, transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.update({ where: { id }, data });
};
```

### Update Many

```javascript
export const updateQuantities = async ({ workOrderId, scrapped, order }, transaction = null) => {
  const Module = transaction?.item ?? Items;
  await Module.updateMany({
    where: {
      workOrderId,
      order: { gt: order },
      parentRouteId: null,
    },
    data: { quantity: { increment: -scrapped } },
  });
};
```

### Delete (soft delete)

```javascript
export const softDelete = async (id, transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.update({
    where: { id },
    data: { deleted: true },
  });
};
```

### Delete (hard delete)

```javascript
export const hardDelete = async (id, transaction = null) => {
  const Module = transaction?.item ?? Items;
  return await Module.delete({ where: { id } });
};
```

## Reusable Select Objects

For models queried from multiple places, define a base select object:

```javascript
const baseItemSelect = {
  id: true,
  name: true,
  status: true,
  customer: { select: { id: true, name: true } },
};

export const findAll = async () =>
  Items.findMany({ select: baseItemSelect });

export const findById = async (id) =>
  Items.findUnique({
    where: { id },
    select: {
      ...baseItemSelect,
      history: { select: { id: true, type: true } },
    },
  });
```

## Rules

- **One PrismaClient per file**, instantiated at module level
- **Never instantiate PrismaClient inside a function**
- **Prefer `select` over `include`** for explicit field selection (avoids over-fetching)
- **Every function ends with `transaction = null`** (or `transaction`) as the last param
- **First line resolves module:** `const Module = transaction?.modelName ?? DefaultModel`
- **Return Prisma result directly** — no try/catch wrapping (services handle errors)
- **Keep queries focused** — create separate repository functions for different select shapes rather than one function with flags
- **No business logic** in repositories — that belongs in the service
