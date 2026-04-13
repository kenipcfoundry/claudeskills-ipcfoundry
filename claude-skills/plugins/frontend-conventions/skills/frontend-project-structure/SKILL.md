---
name: frontend-project-structure
description: Overall project structure, tech stack, folder conventions, and barrel export patterns for the frontend repository React SPA.
---

# Frontend Project Structure

## Tech Stack

- **Framework:** React 17 with React Router DOM 6
- **State:** Redux (global UI) + React Query 3 (server state) + Context (page-level)
- **Forms:** React Hook Form 7 with `FormProvider` / `useFormContext` / `useController`
- **UI:** Material UI 5 (MUI) with custom theme
- **Styling:** MUI `styled()` + `sx` prop (CSS-in-JS via Emotion)
- **Icons:** react-feather + custom SVGs wrapped in MUI `SvgIcon` via `HubIcon`
- **Drag-Drop:** react-beautiful-dnd via `DraggableWrapper`
- **HTTP:** Axios with cookie-based auth (`withCredentials: true`)
- **Dates:** dayjs (UTC mode)
- **Toasts:** notistack via `useHubSnackBar` hook
- **Rich Text:** Tiptap editor

## Directory Layout

```
src/
  App.js                    # Route protection, AuthenticatedUser wrapper
  index.js                  # Provider stack: LocalizationProvider > Redux > Theme > Snackbar > QueryClient > Router
  theme.js                  # MUI theme: colors, typography, component overrides
  routes.js                 # Route definitions: { path, title, element, authenticate }

  components/               # 105 reusable UI components (folder per component)
    index.js                # Barrel export for all components

  pages/                    # 33 page components
    [PageName]/
      [PageName].jsx        # Main page (wraps in PageContent, uses SideNavBarV2)
      [PageName]Bar.jsx     # Page toolbar/header
      helper.js             # Tab definitions, view configs, constants
      styles.js             # Styled components
      index.js              # Barrel export

  containers/               # 42 stateful feature components
    [ContainerName]/
      [ContainerName].jsx   # Redux-connected, data fetching, business logic
      index.js

  tables/                   # 51 table components
    [TableName]/
      [TableName].jsx       # HubTableTemplate with items/keys
      helper.js             # items[] and keys[] arrays
      styles.js
      index.js
    index.js                # Barrel export for all tables

  dialogs/                  # 39 dialog/drawer components
    [DialogName]/
      [DialogName].jsx      # Drawer or Dialog, Redux-connected
      [DialogName]Bar.jsx   # Dialog header
      [DialogName]Details.jsx # Form fields
      helper.js / styles.js / index.js
    index.js                # Barrel export for all dialogs

  hooks/
    react-query/            # 38 data hooks (one per API resource)
    contexts/               # 3 context providers
    useSnackBar.js          # Toast notification hook
    useDebounce.js          # Debounce hook

  redux/                    # Global UI state (auth, dialogs, dark mode)
  constants/                # Permissions, menu items, stage enums
  helpers/                  # Date, currency, cache invalidation utilities
  utils/                    # Axios API client, validation regex
```

## Component Folder Convention

Every component, page, table, dialog, and container follows this structure:

```
ComponentName/
  ComponentName.jsx    # Main component
  helper.js            # Config arrays, utility functions, constants
  styles.js            # styled() components
  index.js             # Barrel: export { ComponentName } from "./ComponentName"
```

Keep files small and focused. Extract `items`/`keys` arrays and config objects into `helper.js`. Extract styled components into `styles.js`. If a sub-component grows beyond ~30 lines, extract it to its own file in the same folder.

## Barrel Exports

All major directories have an `index.js` that re-exports everything:

```javascript
// src/components/index.js
export { HubHookFormInput } from "./HubHookFormInput";
export { HubHookFormSelect } from "./HubHookFormSelect";
export { HubIcon } from "./HubIcon";

// src/tables/index.js
export { ThroughputTable } from "./ThroughputTable";
```

This enables clean imports: `import { HubHookFormInput, HubIcon } from "../../components"` instead of deep paths.

## Routing

```javascript
// routes.js
export const routes = [
  { path: "/signin", title: "Sign In", element: <SignIn /> },
  { path: "/customers/:id", title: "Customer", element: <ContactPage />, authenticate: true },
  { path: "/works/:id", title: "Work Order", element: <WorkOrderDetails />, authenticate: true, useTitleId: true },
  { path: "/sales", title: "Sales", element: <SalesPage />, authenticate: true },
];
```

Routes with `authenticate: true` are wrapped in `AuthenticatedUser` + `TopNavBar` + `ErrorBoundary`.

## Menu & Permissions

Menu items in `constants/menu.js` are filtered by user permissions:

```javascript
{ label: "Sales", path: "/sales", permission: PERMISSIONS_KEY.SALES }
```

`getUserMenuItems(permissions)` filters items where the user's permission value >= VIEW_ONLY (1).

## API Client

```javascript
// utils/api.js — Axios with cookie-based auth
const api = axios.create({ baseURL: config.apiHost, withCredentials: true });

// Response interceptor extracts .data, redirects 401 to /signin
api.interceptors.response.use(
  (response) => response?.data,
  (error) => {
    if (error.response?.status === 401) window.location.href = "/signin";
    return Promise.reject(error);
  }
);
```

## Helpers

```javascript
// Date formatting
import { getDateData } from "../../helpers/dateHelper";
getDateData({ startDate: value, includeYear: true });  // "Apr 13, 2026"

// Currency formatting
import { getDollarValue } from "../../helpers/currency";
getDollarValue(1234.5);  // "$1,234.50"

// React Query cache invalidation
import { invalidate } from "../../helpers/clientHelper";
invalidate(queryClient, "/schedule");  // invalidates all /schedule* queries
```

## Snackbar Notifications

```javascript
const { addSnackbarSuccess, addSnackbarError } = useHubSnackBar();
addSnackbarSuccess({ message: "Successfully created item" });
addSnackbarError({ error });  // extracts message from axios error
```
