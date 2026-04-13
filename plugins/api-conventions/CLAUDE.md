# Backend API Project Standards

This project follows company coding conventions. Always reference and apply the
appropriate skill for every task, regardless of how the request is phrased.

## Skill Usage Rules

When building or modifying ANY of the following, always load and follow the
corresponding skill before writing any code:

- Any Express route, endpoint, or route handler
  → use `/api-conventions:api-controllers`

- Any business logic, transaction, or data operation
  → use `/api-conventions:api-services`

- Any database query or Prisma operation
  → use `/api-conventions:api-repositories`

- Any request body or query parameter validation
  → use `/api-conventions:api-schemas`

- Any error throwing, catching, or exception handling
  → use `/api-conventions:api-errors`

- Any Prisma schema change, new model, relation, or migration
  → use `/api-conventions:api-prisma`

- Any utility function, calculation, grouping, or data transformation
  → use `/api-conventions:api-helpers-mappers`

- Any real-time update or SSE broadcast
  → use `/api-conventions:api-sse`

- Creating a new module, feature, or folder structure
  → use `/api-conventions:api-project-structure`

## General Rules

- Always apply these conventions even if the user just says "build this",
  "add a feature", or "fix this bug"
- Never write Prisma queries directly in services — use repository functions
- Never write business logic in controllers — use services
- Always wrap service functions in try/catch
- Always re-throw NotFoundException and BadRequestException
- Always use prisma.$transaction() for multi-step writes
- Always broadcast SSE after the transaction commits, never inside it
