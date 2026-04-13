---
name: api-prisma
description: Prisma schema conventions for the backend repository PostgreSQL database. Covers model naming, field types, relations, mapping, cascade deletes, and migration workflow.
---

# API Prisma Schema

## Configuration

```prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["multiSchema"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
  schemas  = ["quotes"]
}
```

## Model Conventions

```prisma
model Lead {
  id          Int       @id @default(autoincrement())
  customerId  Int?      @map("customer_id")
  companyName String?   @map("company_name")
  stage       Int       @default(0)
  value       Decimal?  @db.Decimal(10, 2)
  closedAt    DateTime? @map("closed_at")
  createdById Int       @map("created_by_id")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")

  // Relations
  customer    Customer? @relation(fields: [customerId], references: [id])
  createdBy   User      @relation("LeadCreator", fields: [createdById], references: [id])
  assignee    User?     @relation("LeadAssignee", fields: [assigneeId], references: [id])
  history     LeadHistory[]
  quotes      SalesQuote[]

  @@map("leads")
  @@schema("quotes")
}

model LeadHistory {
  id        Int      @id @default(autoincrement())
  leadId    Int      @map("lead_id")
  userId    Int      @map("user_id")
  type      String?
  fromStage Int?     @map("from_stage")
  toStage   Int?     @map("to_stage")
  value     Json?
  createdAt DateTime @default(now()) @map("created_at")

  lead      Lead     @relation(fields: [leadId], references: [id], onDelete: Cascade)
  user      User     @relation("LeadHistoryUser", fields: [userId], references: [id])

  @@map("lead_history")
  @@schema("quotes")
}
```

## Naming Rules

| What | Convention | Example |
|------|-----------|---------|
| Model name | PascalCase | `Lead`, `SalesQuote`, `WorkOrderRouting` |
| Field name (Prisma) | camelCase | `customerId`, `companyName`, `createdAt` |
| Column name (DB) | snake_case via `@map()` | `@map("customer_id")`, `@map("company_name")` |
| Table name (DB) | snake_case via `@@map()` | `@@map("leads")`, `@@map("sales_quotes")` |
| Schema | Always `@@schema("quotes")` | `@@schema("quotes")` |
| Relation name | Quoted string when model has multiple relations to same target | `@relation("LeadCreator")` |

## Field Types

| Prisma Type | Usage | Decorators |
|-------------|-------|------------|
| `Int` | IDs, counts, stages | `@id @default(autoincrement())` |
| `String` | Text fields | `@unique`, `@db.VarChar(50)` |
| `String?` | Nullable text | `@map("snake_case")` |
| `Boolean` | Flags | `@default(true)` |
| `DateTime` | Created timestamps | `@default(now()) @map("created_at")` |
| `DateTime` | Updated timestamps | `@updatedAt @map("updated_at")` |
| `DateTime?` | Optional dates | `@map("closed_at")` |
| `Decimal` | Currency/precision | `@db.Decimal(10, 2)` |
| `Float` | Approximate numbers | |
| `Json` | Flexible data | History change values |
| `Int?` | Nullable FK | `@map("customer_id")` |

## Relations

### One-to-Many
```prisma
// Parent side (one)
model Customer {
  leads Lead[]
}

// Child side (many)
model Lead {
  customerId Int?     @map("customer_id")
  customer   Customer? @relation(fields: [customerId], references: [id])
}
```

### Multiple Relations to Same Model
```prisma
model Lead {
  assigneeId  Int?  @map("assignee_id")
  createdById Int   @map("created_by_id")
  assignee    User? @relation("LeadAssignee", fields: [assigneeId], references: [id])
  createdBy   User  @relation("LeadCreator", fields: [createdById], references: [id])
}

model User {
  assignedLeads Lead[] @relation("LeadAssignee")
  createdLeads  Lead[] @relation("LeadCreator")
}
```

### Cascade Delete
```prisma
// Child deleted when parent is deleted
lead Lead @relation(fields: [leadId], references: [id], onDelete: Cascade)

// FK set to null when parent is deleted
customer Customer? @relation(fields: [customerId], references: [id], onDelete: SetNull)
```

## Migration Workflow

```bash
# Create migration
npx prisma migrate dev --name add_feature_name

# Reset database (test environments)
npx prisma migrate reset --force

# Generate client after schema changes
npx prisma generate
```
