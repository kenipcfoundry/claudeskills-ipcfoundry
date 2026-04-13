---
name: api-controllers
description: How to write Express route controllers in the backend repository. Controllers export arrays of route objects with path, method, authentication, permissions, Joi validation, and async action handlers.
---

# API Controllers

Controllers export an **array of route objects**. Each object defines a single endpoint with its full middleware config.

## Route Object Shape

```javascript
{
  path: string,           // URL path: "/labor", "/works/:id"
  method: string,         // "get" | "post" | "put" | "delete"
  authenticate: boolean,  // true = requires JWT token in cookies
  permission: number,     // PERMISSIONS_KEY value (from constants/permissions.js)
  validate?: object,      // Joi schema for request body (POST/PUT)
  query?: object,         // Joi validators for query parameters (GET)
  action: async (req, res) => void
}
```

## Full Controller Example

```javascript
import Joi from "joi";
import { createItem, getItems, updateItem, deleteItem } from "./ItemService.js";
import { ItemCreateSchema, ItemUpdateSchema } from "./ItemSchema.js";
import { PERMISSIONS_KEY } from "../../constants/permissions.js";

const prefix = "/items";

export default [
  // GET with query param validation
  {
    path: `${prefix}`,
    method: "get",
    authenticate: true,
    permission: PERMISSIONS_KEY.ITEMS,
    query: {
      location: Joi.number().integer().min(1).max(3).required()
        .error(new Error("Invalid Location")),
      orderBy: Joi.string().valid("name", "createdAt").required()
        .error(new Error("Invalid Order By")),
      sortOrder: Joi.string().valid("desc", "asc").required()
        .error(new Error("Invalid Sort Order")),
      limit: Joi.number().valid(10, 25, 50, 75, 100).required()
        .error(new Error("Invalid Limit")),
      offset: Joi.number().required().error(new Error("Invalid Offset")),
    },
    action: async (req, res) => {
      try {
        const locationId = Number(req.query.location);
        const pagination = {
          orderBy: req.query.orderBy,
          sortOrder: req.query.sortOrder,
          limit: parseInt(req.query.limit),
          offset: parseInt(req.query.offset),
        };
        const results = await getItems({ locationId, pagination });
        res.status(200).json(results);
      } catch (err) {
        res.status(400).send(err);
      }
    },
  },

  // GET with URL param
  {
    path: `${prefix}/:id`,
    method: "get",
    authenticate: true,
    permission: PERMISSIONS_KEY.ITEMS,
    action: async (req, res) => {
      try {
        const id = Number(req.params.id);
        const results = await getItemById(id);
        res.status(200).json(results);
      } catch (err) {
        res.status(400).send(err);
      }
    },
  },

  // POST with imported Joi schema
  {
    path: `${prefix}`,
    method: "post",
    authenticate: true,
    permission: PERMISSIONS_KEY.ITEMS,
    validate: ItemCreateSchema,
    action: async (req, res) => {
      try {
        const userId = Number(req.user.id);
        const payload = { ...req.body, createdById: userId };
        const results = await createItem(payload);
        res.status(200).json(results);
      } catch (err) {
        res.status(400).send(err);
      }
    },
  },

  // PUT with imported Joi schema and URL param
  {
    path: `${prefix}/:id`,
    method: "put",
    authenticate: true,
    permission: PERMISSIONS_KEY.ITEMS,
    validate: ItemUpdateSchema,
    action: async (req, res) => {
      try {
        const id = Number(req.params.id);
        const userId = Number(req.user.id);
        const results = await updateItem(id, req.body, userId);
        res.status(200).json(results);
      } catch (err) {
        res.status(400).send(err);
      }
    },
  },

  // DELETE
  {
    path: `${prefix}/:id`,
    method: "delete",
    authenticate: true,
    permission: PERMISSIONS_KEY.ITEMS,
    action: async (req, res) => {
      try {
        const id = Number(req.params.id);
        const results = await deleteItem(id);
        res.status(200).json(results);
      } catch (err) {
        res.status(400).send(err);
      }
    },
  },
];
```

## Rules

- `query` validates URL query parameters (GET requests)
- `validate` validates request body (POST/PUT requests)
- Both can be inline Joi validators or imported schema objects
- `action` is always `async (req, res) => { try/catch }`
- User info available via `req.user.id` (set by auth middleware)
- Always return `res.status(200).json(data)` on success
- Always return `res.status(400).send(err)` on error
- Keep controller logic minimal: extract params, call service, return response
- No business logic in controllers — that belongs in the service
- No Prisma queries in controllers — that belongs in the repository
