---
name: api-errors
description: Custom exception classes and error handling patterns in the backend repository. Covers BadRequestException, NotFoundException, InternalServerException, and how they propagate through the service/controller layers.
---

# API Error Handling

## Custom Exception Classes

All exceptions live in `errors/` and extend `Error`:

```javascript
export const BadRequestException = class extends Error {
  constructor(message, code) {
    super();
    this.message = message;
    this.type = "BadRequest";
    this.code = code || 400;
  }
};
```

## Available Exceptions

| Class | Code | When to Use |
|-------|------|-------------|
| `BadRequestException` | 400 | Invalid input, business rule violation, quantity exceeded |
| `NotFoundException` | 404 | Entity not found by ID |
| `ForbiddenException` | 403 | Permission denied (used by middleware) |
| `UnauthorizedException` | 401 | Invalid/expired auth token (used by middleware) |
| `InternalServerException` | 500 | Unexpected errors (catch-all wrapper) |
| `ConflictException` | 409 | Duplicate/conflict (e.g., unique constraint violation) |

Import from `../../errors/index.js`.

## Error Handling in Services

```javascript
import {
  BadRequestException,
  InternalServerException,
  NotFoundException,
} from "../../errors/index.js";

export const updateItem = async (id, data) => {
  try {
    const item = await findById(id);
    if (!item) throw new NotFoundException(`Item ${id} not found`);

    if (data.quantity < item.completed) {
      throw new BadRequestException("Cannot reduce quantity below completed amount");
    }

    return await updateRecord({ id, data });
  } catch (e) {
    console.log(e);
    // Re-throw user-facing exceptions
    if (e instanceof NotFoundException || e instanceof BadRequestException) throw e;
    // Wrap everything else
    throw new InternalServerException("Failed to update item");
  }
};
```

## Error Handling in Controllers

```javascript
action: async (req, res) => {
  try {
    const results = await serviceFunction(payload);
    res.status(200).json(results);
  } catch (err) {
    res.status(400).send(err);
  }
},
```

The exception's `message`, `type`, and `code` properties are sent to the client via `res.send(err)`.

## Rules

- **Services** throw `BadRequestException` and `NotFoundException` for known error conditions
- **Services** always re-throw these two (they carry meaningful user messages)
- **Services** catch everything else and wrap in `InternalServerException`
- **Controllers** catch and return `res.status(400).send(err)` (the exception object serializes to JSON)
- **Always `console.log(e)`** before re-throwing for server-side debugging
- **Never expose** raw Prisma errors or stack traces to the client
