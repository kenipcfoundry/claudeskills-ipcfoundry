---
name: frontend-styling
description: MUI theme configuration, styled() component patterns, color system, typography variants, and spacing conventions for the frontend repository.
---

# Frontend Styling & Theme

## styled() Components

All styling uses MUI's `styled()` function in `styles.js` files:

```javascript
import { styled } from "@mui/material/styles";
import { Box, Drawer, TableRow } from "@mui/material";

// Basic styled component
export const StyledDrawer = styled(Drawer)({
  "& .MuiDrawer-paper": {
    width: "100%",
    maxWidth: "600px",
  },
});

// With theme access
export const StyledTableRow = styled(TableRow)(({ theme }) => ({
  backgroundColor: "transparent",
  "&:hover": {
    backgroundColor: theme.palette.hubColors.greyLighter,
  },
}));

// With custom props
export const StyledTableRow = styled(TableRow)(({ theme, main }) => ({
  backgroundColor: main ? theme.palette.hubColors.greyLightest : "transparent",
  borderBottom: `1px solid ${theme.palette.hubColors.greyLight}`,
}));

// With nested selectors
export const StyledBox = styled(Box)({
  width: 130,
  "& .MuiInputBase-root": {
    fontSize: "12px",
  },
  "& .MuiOutlinedInput-input": {
    padding: "6px 10px",
  },
});
```

## Theme Colors

Access via `theme.palette.hubColors.*` in styled components or `sx` prop:

```
mainFocus         — primary orange: rgb(223, 158, 68)
mainFocusDark     — dark orange: rgb(210, 140, 42)
redError          — #F81414
greenMain         — rgb(0, 193, 45)
blueMain          — rgb(0, 117, 255)
yellowMain        — rgb(255, 183, 0)
white             — #FFFFFF
greyLightest      — #F7F7F7
greyLighter       — #F4F4F4
greyLight         — #E5E5E5
greyMid           — #C0C0C0
grey              — #AEAEAE
greyDark          — #525252
greyDark2         — #252525
black             — #000000
```

Usage in `sx` prop (shorthand, no `theme.palette`):

```jsx
<Box sx={{ color: "hubColors.greyDark", backgroundColor: "hubColors.greyLightest" }} />
```

Usage in `styled()` (requires `theme.palette`):

```javascript
styled(Box)(({ theme }) => ({
  color: theme.palette.hubColors.greyDark,
}));
```

## Typography Variants

Custom variants available on `<Typography variant="...">`:

| Variant | Size | Weight | Color | Use |
|---------|------|--------|-------|-----|
| `caption` | 12px | 400 | greyDark | Default small text |
| `caption11` | 11px | 400 | greyDark | Compact table cells |
| `captionGrey` | 12px | 400 | grey | Muted text |
| `boldCaption` | 12px | 700 | uppercase | Section labels |
| `boldCaptionMain` | 12px | 700 | mainFocus (orange) | Highlighted labels |
| `boldCaptionMainL` | 12px | 700 | blueMain | Blue links |
| `darkBold14` | 14px | 500 | black | Emphasis text |
| `body1` | 16px | 400 | greyDark | Body text |
| `body2` | 14px | 400 | greyDark | Smaller body text |
| `h3` | 24px | 800 | — | Page titles |

```jsx
<Typography variant="caption" fontSize={11}>Small text</Typography>
<Typography variant="boldCaptionMainL" fontSize={12}>Blue link text</Typography>
```

## Spacing

`theme.spacing(1)` = 5px base unit.

```jsx
// In sx prop
<Box sx={{ p: "15px", gap: "10px", m: 2 /* = 10px */ }} />

// In styled
styled(Box)(({ theme }) => ({
  padding: theme.spacing(3),  // 15px
}));
```

## Common Patterns

### Table Cell Variants

```jsx
<TableCell variant="blueBold">  // blue text, bold, clickable
<TableCell variant="small">     // compact cell
```

### Stack Layouts

```jsx
// Row layout
<Stack direction="row" gap="10px" alignItems="center">

// Column layout with scroll
<Stack gap="20px" overflow="auto" flex={1}>
```

### sx Prop Usage

```jsx
<Box sx={{
  width: "200px",
  height: "18px",
  cursor: "pointer",
  "&:hover": { backgroundColor: "hubColors.greyLighter" },
}} />
```

## Rules

- Use `styled()` from `@mui/material/styles` (not `@emotion/styled` directly)
- Put all styled components in `styles.js` in the same folder
- Use `sx` prop for one-off inline styles
- Use `styled()` for reusable styled components
- Access theme colors via `theme.palette.hubColors.*` in styled, `"hubColors.*"` in sx
- Never use raw CSS files — everything is CSS-in-JS
