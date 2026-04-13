---
name: api-project-structure
description: Overall project structure, tech stack, layered architecture, and module organization for the backend repository Node.js Express REST API.
---

# API Project Structure

## Tech Stack

- **Runtime:** Node.js with Express
- **Database:** PostgreSQL via Prisma ORM (`@prisma/client`)
- **Validation:** Joi (`joi`)
- **Auth:** JWT (`jsonwebtoken`) + bcryptjs, cookie-based (`cookie-parser`)
- **Dates:** dayjs, date-fns
- **Cloud:** AWS SDK (S3 file storage, SES email)
- **File handling:** Multer (uploads), exceljs (Excel export)
- **Logging:** Morgan
- **Rate limiting:** express-rate-limit
- **Testing:** Jest + Chai (integration tests)
- **API docs:** Swagger (swagger-jsdoc + swagger-ui-express)

## Directory Layout

```
backend/
  server.js                 # Express server entry + SSE endpoints
  src/
    index.js                # Router setup, middleware chain, route registration
    middleware.js            # authenticateToken, validatePayload, validateQuery, validatePermission
    routes.js               # Imports all controllers, exports flat routes array
    sse/displayBus.js       # SSE event bus singleton for real-time updates
    [module]/               # Feature modules (labor, workOrders, sales, etc.)
      [Module]Controller.js # Route definitions (array of route objects)
      [Module]Service.js    # Business logic, transactions, error handling
      [Module]Repository.js # Prisma queries, data access
      [Module]Schema.js     # Joi validation schemas
      [Module]Helper.js     # Utility/calculation functions
      [Module]Mapper.js     # DB-to-API response transformations
      [Module].integration.test.js
  constants/
    permissions.js          # Permission groups, types, keys (PERMISSIONS_KEY)
    salesStages.js          # Lead/Quote stage enums and labels
    Jobs.js                 # Job statuses, departments
    index.js                # Re-exports all constants
  errors/
    BadRequestException.js  # 400
    NotFoundException.js    # 404
    ForbiddenException.js   # 403
    UnauthorizedException.js # 401
    InternalServerException.js # 500
    ConflictException.js    # 409
    index.js                # Re-exports all exceptions
  helpers/
    pagination.js           # Joi pagination schemas + getJoiPaginationBody()
    date.js                 # Date formatting utilities
    currency.js             # Currency formatting
    encryption.js           # Encrypt/decrypt helpers
  prisma/
    schema.prisma           # Database schema (PostgreSQL, multi-schema "quotes")
    migrations/             # Prisma migrations
```

## Architecture: Layered Request Flow

```
HTTP Request
  -> middleware.js (authenticateToken -> validatePayload/validateQuery -> validatePermission)
  -> Controller (route handler, extracts params, calls service)
  -> Service (business logic, transactions, calls repository)
  -> Repository (Prisma queries, optional transaction parameter)
  -> Database (PostgreSQL)
```

## Routes Registration

All controllers are imported and flattened in `src/routes.js`:

```javascript
import labor from "./labor/LaborController.js";
import works from "./workOrders/WorkOrderController.js";

export const routes = [labor, works, /* ... */].flat();
```

The router in `src/index.js` iterates routes and builds the middleware chain dynamically:

```javascript
routes.forEach(({ path, method, authenticate, validate, query, permission, action }) => {
  const middlewares = [
    authenticate && authenticateToken(authenticate),
    validate && validatePayload(validate),
    query && validateQuery(query),
    permission && validatePermission(permission, accessType),
  ].filter(Boolean);

  router[method](path, middlewares, action);
});
```

## Constants

```javascript
// permissions.js
export const PERMISSION_GROUP_TYPES = { CUSTOMERS: 0, ORDERS: 1, /* ... */ SALES: 9 };
export const PERMISSIONS = [
  { id: 1, label: "CUSTOMERS", type: PERMISSION_TYPES.ACCESS, group: 0 },
  { id: 31, label: "SALES", type: PERMISSION_TYPES.ACCESS, group: 9 },
];
export const PERMISSIONS_KEY = { CUSTOMERS: 1, TICKETS: 18, SALES: 31 };

// salesStages.js
export const LEAD_STAGES = { PROSPECTING: 0, QUALIFYING: 1, /* ... */ LOST: 5 };
export const LEAD_STAGE_LABELS = [{ id: 0, label: "Prospecting" }, /* ... */];
export const HISTORY_TYPES = { STAGE_CHANGE: "stage_change", FIELD_UPDATE: "field_update", CREATION: "creation" };

// Jobs.js
export const JOB_STATUSES = { NOT_STARTED: "Not Started", IN_PROGRESS: "In Progress", COMPLETE: "Completed" };
```

## Pagination

Standardized via `helpers/pagination.js`:

```javascript
// Controller query validation
query: {
  ...getJoiPaginationProps({ valid: ["workOrder", "startDate"] }),
}

// Controller body extraction
const pagination = getJoiPaginationBody(req.query);
// Returns: { orderBy, sortOrder, limit: parseInt(limit), offset: parseInt(offset) }

// Service response format
return { rows: mappedData, meta: { count, pages, limit, offset, sortOrder, orderBy } };
```

## Testing

Integration tests use Jest, call service functions directly against a test database:

```javascript
describe("Feature", () => {
  test("Description", async () => {
    const data = await createTestData();
    const result = await serviceFunction(payload);
    expect(result.field).toEqual(expected);
  });
});
```

Run: `npm run test:local` (spins up Docker DB, resets migrations, runs Jest).
