---
name: frontend-forms
description: React Hook Form integration patterns for the frontend repository. Covers HubHookFormWrapper, HubHookFormInput, HubHookFormSelect, HubHookFormSelectV2, HubHookFormDatePicker, and HubHookFormTagInput.
---

# Frontend Forms (React Hook Form)

## HubHookFormWrapper

Wraps a page/dialog section in `FormProvider`. Auto-resets when `defaultValues` change (e.g., after API refetch):

```jsx
<HubHookFormWrapper
  defaultValues={isNew ? {} : { ...data }}
  onSubmit={(data, event, dirtyFields) => handleSubmit(data)}
>
  <HubHookFormInput name="companyName" required />
  <HubHookFormSelect name="locationId" options={FOUNDRIES} />
  <HubButton type="submit" label="Save" />
</HubHookFormWrapper>
```

Props:
- `defaultValues` — initial form values (reset when changed)
- `onSubmit(data, event, dirtyFields)` — form submit handler
- `useReset` — auto-reset on defaultValues change (default: true)

## HubHookFormInput

Text input with built-in formatting and validation:

```jsx
<HubHookFormInput
  name="quantity"           // react-hook-form field name (required)
  defaultValue={100}
  placeholder="Enter qty"
  label="Quantity"
  required                  // or required="Custom error message"
  disableHelperText         // suppress validation message below field
  isNumber                  // numeric input, strips non-digits
  isCurrency                // decimal + $ adornment
  isDecimal                 // decimal number input
  isEmail                   // email validation regex
  isNumerical               // strips non-digits on input
  isUppercase               // toUpperCase on input
  onBlur={(e, value) => handleBlur(value)}
  onChange={(e, value) => handleChange(value)}
  hidden                    // visually hidden but registered in form
  rows={3}                  // textarea mode
  startAdornment={<HubIcon icon="Search" />}
  endAdornment={<HubIcon icon="X" />}
/>
```

**Key behaviors:**
- `isNumber`: sets input type="number", strips non-digits, clears "0" on focus
- `isCurrency`: decimal pattern + $ prefix adornment via `getCurrencySymbol(currency)`
- `onBlur(e, value)`: receives the formatted value, not the raw event
- `hidden`: `display: none` but still registered in the form
- Uses `useController` internally for form integration

## HubHookFormSelect

Standard dropdown (MUI Select). For small fixed-option lists:

```jsx
<HubHookFormSelect
  name="locationId"
  options={[{ id: 1, label: "Utah" }, { id: 2, label: "Oregon" }]}
  defaultValue={1}
  required
  disableHelperText
  onChange={(e) => handleChange(e.target.value)}
  multiple                  // allow multiple selections
  allowNull                 // include empty/null option
  RenderComponent={CustomOption}  // custom option renderer
/>
```

## HubHookFormSelectV2

Searchable autocomplete (MUI Autocomplete). For large/dynamic lists:

```jsx
<HubHookFormSelectV2
  name="partId"
  options={partOptions}          // [{ id, label, ...extra }]
  RenderComponent={({ label, status }) => <CustomOption label={label} status={status} />}
  renderValue={(value) => <SelectedDisplay value={value} />}
  onInputChange={(searchText) => debouncedSearch(searchText)}
  placeholder="Search parts..."
  required
  useSetValue               // also call setValue on change
  emptyOption={{            // shown when no results match
    label: "Create new part",
    onSelect: (inputValue) => createPart(inputValue),
  }}
/>
```

**Key differences from V1:**
- Uses MUI Autocomplete (text input with filtering) instead of Select (dropdown)
- Supports `onInputChange` for search-as-you-type
- Supports `emptyOption` for "create new" flows
- Better for large datasets

## HubHookFormDatePicker

Date picker with UTC formatting:

```jsx
<HubHookFormDatePicker
  name="startDate"
  defaultValue={item.startDate}
  onChange={(value) => handleDateChange(value)}
  placeholder="MM/DD/YYYY"
  required
  disabled={!part?.id}
  includeTime               // show time picker too
  onOpen={() => {}}         // passed through to MUI DatePicker
  onClose={() => {}}        // passed through to MUI DatePicker
/>
```

**Important:** `onOpen` and `onClose` pass through to MUI DatePicker via `{...rest}`. Useful for portal-aware dismiss logic with `ClickAwayListener`.

## HubHookFormTagInput

Multi-select autocomplete with tags/chips:

```jsx
<HubHookFormTagInput
  name="departments"
  label="Departments"
  options={departmentOptions}     // [{ id, label }]
  defaultValue={[1, 3, 5]}       // array of selected IDs
  required
  multiple                        // allow multiple selections
  allowAdd                        // allow creating new options by typing
  RenderComponent={CustomTag}
/>
```

## HubHookFormSwitch

Toggle switch:

```jsx
<HubHookFormSwitch name="isActive" label="Active" defaultValue={true} />
```

## Form Context Access

Inside any child of `HubHookFormWrapper`, access form state via:

```javascript
const { watch, setValue, getValues, formState: { dirtyFields } } = useFormContext();

const currentValue = watch("fieldName");
setValue("fieldName", newValue, { shouldDirty: true });
const allValues = getValues();
```

## Rules

- Always use `useController` (not `register`) for custom input components
- Never create a hidden `<input {...register(name)} />` alongside `useController` — it causes dual-registration conflicts
- Use `disableHelperText` on table/compact inputs to save space
- `required` can be `true` or a string (custom error message)
- Import all form components from `../../components` barrel
