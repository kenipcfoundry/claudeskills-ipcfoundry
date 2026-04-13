---
name: api-schemas
description: How to write Joi validation schemas in the backend repository. Covers reusable field templates, required vs optional patterns, conditionals, error messages, and schema composition.
---

# API Schemas (Joi Validation)

Schemas define validation rules for request bodies. They live in `[Module]Schema.js` files and are referenced by controllers.

## Reusable Field Templates

Define common field types at the top of each schema file:

```javascript
import Joi from "joi";

const ID = Joi.number().integer().min(1);
const OpID = ID.optional().allow(null);
const OpString = Joi.string().optional().allow(null, "");
const OpFloat = Joi.number().precision(2).optional().allow(null);
const OpInt = Joi.number().integer().optional().allow(null);
const OpBool = Joi.boolean().optional().allow(null);
```

## Schema as Plain Object

Used for the `validate` property in controllers. Each field has `.error(new Error("message"))`:

```javascript
export const ItemUpdateSchema = {
  name: Joi.string().required().error(new Error("Invalid Name")),
  price: OpFloat.error(new Error("Invalid Price")),
  quantity: Joi.number().min(1).optional().error(new Error("Invalid Quantity")),
  notes: OpString,
};
```

## Schema as Joi.object

Used when you need conditional validation, `.xor()`, or other Joi.object features:

```javascript
export const ItemCreateSchema = Joi.object({
  locationId: ID.required().error(new Error("Invalid Location Id")),
  isRepeat: Joi.boolean().required(),

  // Conditional: only allowed when isRepeat is true
  repeatOption: Joi.when("isRepeat", {
    is: true,
    then: Joi.number().integer().min(1).max(4).required(),
    otherwise: Joi.forbidden(),
  }),

  // Nested conditional
  repeatWeekdays: Joi.when("isRepeat", {
    is: true,
    then: Joi.array()
      .items(Joi.number().integer().min(0).max(6))
      .when("repeatOption", {
        is: 4,
        then: Joi.array().min(1).max(7).required(),
        otherwise: Joi.optional(),
      }),
    otherwise: Joi.forbidden(),
  }),
});
```

## Shared Field Sets

Define common fields once, spread into create/update schemas:

```javascript
const quoteFields = {
  customerId: OpID.error(new Error("Invalid Customer Id")),
  partId: OpID.error(new Error("Invalid Part Id")),
  weight: OpFloat.error(new Error("Invalid Weight")),
  price: OpFloat.error(new Error("Invalid Price")),
  notes: OpString,
};

export const QuoteCreateSchema = Joi.object({
  locationId: ID.required().error(new Error("Invalid Location Id")),
  ...quoteFields,
});

export const QuoteUpdateSchema = Joi.object({
  locationId: OpID.error(new Error("Invalid Location Id")),
  ...quoteFields,
  lostReason: OpString.error(new Error("Invalid Lost Reason")),
});
```

## Nested Object Validation

```javascript
export const UpdateSchema = {
  laborMaterials: Joi.array()
    .items(
      Joi.object({
        materialId: Joi.number().integer().required().min(1)
          .error(new Error("Invalid Material Id")),
        quantity: Joi.number().precision(2).required()
          .error(new Error("Invalid Quantity")),
      })
    )
    .optional()
    .allow(null)
    .error(new Error("Invalid Labor Materials")),
};
```

## Stage Validation

For enum-like integer stages:

```javascript
export const StageUpdateSchema = Joi.object({
  stage: Joi.number().integer().min(0).max(5).required()
    .error(new Error("Invalid stage")),
});
```

## Rules

- Every field has `.error(new Error("message"))` for clear validation messages
- Required fields: `.required()`
- Optional nullable: `.optional().allow(null)`
- Optional nullable string: `.optional().allow(null, "")`
- Decimal numbers: `.precision(2)`
- Enums: `.valid("value1", "value2")` or `.min(0).max(5)`
- Conditionals: `.when("field", { is, then, otherwise })`
- ISO dates: `.isoDate()` (validates ISO 8601 format)
- Define reusable field templates (OpID, OpString, etc.) at top of file
- Spread shared fields: `{ ...quoteFields }` into specific schemas
- Use `Joi.object({})` only when you need Joi.object-level features (conditionals, `.xor()`, etc.)
- Use plain object `{}` for simple field-by-field validation
