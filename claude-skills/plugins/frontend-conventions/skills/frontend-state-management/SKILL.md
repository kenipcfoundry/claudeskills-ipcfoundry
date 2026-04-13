---
name: frontend-state-management
description: Redux, React Context, and React Query state management patterns for the frontend repository. Covers when to use each, Redux auth/dialogs slices, context providers, and SSE real-time updates.
---

# Frontend State Management

## When to Use What

| State Type | Tool | Examples |
|-----------|------|----------|
| **Global UI** | Redux | Auth user, permissions, dialog open/close, dark mode |
| **Page-level UI** | Context | Active tab, view mode, local toggles |
| **Server data** | React Query | API data, caching, fetching, mutations, invalidation |

## Redux

### Auth Slice (`redux/auth.js`)

```javascript
// State shape
auth: {
  user: {
    id: 1,
    role: "admin",
    permissions: [{ id: 1, value: 4 }, { id: 31, value: 3 }],
    locationId: 1,
    locations: [{ id: 1, name: "Utah" }],
  },
  darkMode: false,
}

// Selectors
import { getAuthUser, getUserPermissions, getUserLocation } from "../../redux/auth";

const user = useSelector(getAuthUser);
const permissions = useSelector(getUserPermissions);
const locationId = useSelector(getUserLocation);

// Permission check
import { getUserPermissionById } from "../../redux/auth";
import { PERMISSIONS_KEY, PERMISSION_LEVEL_KEYS } from "../../constants/permissions";

const canEdit = useSelector((state) =>
  getUserPermissionById(state, PERMISSIONS_KEY.CUSTOMERS, PERMISSION_LEVEL_KEYS.EDIT)
);
```

### Dialogs Slice (`redux/dialogs.js`)

```javascript
// State shape
dialogs: {
  LeadDialog: false,        // closed
  QuoteDialog: 42,          // open editing item 42
  CreateDialog: true,       // open in create mode
}

// Actions
import { setIsOpenDialog } from "../../redux/dialogs";

dispatch(setIsOpenDialog("LeadDialog", true));     // open create
dispatch(setIsOpenDialog("LeadDialog", 42));        // open edit
dispatch(setIsOpenDialog("LeadDialog", false));     // close

// Selectors
import { getIsDialogOpen } from "../../redux/dialogs";

const open = useSelector((state) => getIsDialogOpen(state, "LeadDialog"));
// Returns: false | true | number

// In connected components
const mapStateToProps = (state) => ({
  open: getIsDialogOpen(state, "LeadDialog"),
});
const mapDispatchToProps = { _setIsOpenDialog: setIsOpenDialog };
export default connect(mapStateToProps, mapDispatchToProps)(Component);
```

## Context Providers

For page-level UI state. Three exist: `ScheduleContextProvider`, `ShippingContextProvider`, `WorkOrdersContextProvider`.

```jsx
// Definition (hooks/contexts/ScheduleContextProvider.jsx)
import React, { createContext, useContext, useState } from "react";

const ScheduleContext = createContext();

export const ScheduleContextProvider = ({ children }) => {
  const [open, setOpen] = useState(false);
  const [tab, setTab] = useState(1);

  return (
    <ScheduleContext.Provider value={{ open, setOpen, tab, setTab }}>
      {children}
    </ScheduleContext.Provider>
  );
};

export const useScheduleContext = () => {
  const context = useContext(ScheduleContext);
  if (!context) throw new Error("useScheduleContext must be used within ScheduleContextProvider");
  return context;
};

// Usage in page
const SchedulePage = () => (
  <ScheduleContextProvider>
    <PageContent>
      <SchedulePageContent />
    </PageContent>
  </ScheduleContextProvider>
);

// Usage in child
const { tab, setTab } = useScheduleContext();
```

## SSE (Server-Sent Events)

Two hooks for real-time updates:

### useDisplaySSE — Station Display Updates

```javascript
// Invalidates React Query cache when server pushes update
import { useDisplaySSE } from "../../hooks";

useDisplaySSE(locationId);  // call in component, no return value
```

Internally uses browser `EventSource` with `withCredentials: true`. On receiving `{ type: "update" }`, invalidates `"stationdisplays"` React Query cache.

### useNotificationSSE — Notification Count

```javascript
// Dispatches Redux action for notification badge count
import { useNotificationSSE } from "../../services/notifications/useNotificationSSE";

useNotificationSSE();  // call in component, no return value
```

On `{ type: "init" }`: sets unread count. On `{ type: "notification" }`: increments count. Manual reconnect after 5s on error.

## Provider Stack (index.js)

```jsx
<LocalizationProvider dateAdapter={AdapterDayjs}>
  <Provider store={store}>           {/* Redux */}
    <IpcThemeProvider>                {/* MUI Theme */}
      <SnackbarProvider maxSnack={1}> {/* Notistack */}
        <HubQueryClientProvider>      {/* React Query */}
          <RouterProvider router={router} />
        </HubQueryClientProvider>
      </SnackbarProvider>
    </IpcThemeProvider>
  </Provider>
</LocalizationProvider>
```

## Rules

- **Redux** for state that must persist across page navigation (auth, dialogs)
- **Context** for state scoped to a single page (tabs, view modes)
- **React Query** for all server data — never store API responses in Redux
- Context providers wrap the page component, not the entire app
- Use `connect()` for class-like patterns, `useSelector`/`useDispatch` for hooks
- Reselect (`createSelector`) for memoized selectors that derive data
