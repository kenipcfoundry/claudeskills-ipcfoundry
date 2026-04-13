---
name: frontend-dialogs
description: Dialog and drawer patterns for the frontend repository. Covers Redux-controlled open/close state, MUI Drawer rendering, HubDialogWrapper with context, and form integration.
---

# Frontend Dialogs

## Redux-Controlled Drawer Pattern

Dialogs use Redux for open/close state and MUI Drawer for the UI:

```jsx
import React from "react";
import { connect } from "react-redux";
import { getIsDialogOpen, setIsOpenDialog } from "../../redux/dialogs";
import { useGetLeadDetails, usePostLead, usePutLead } from "../../hooks/react-query/useSalesData";
import { HubHookFormWrapper } from "../../components";
import LeadBar from "./LeadBar";
import LeadDetails from "./LeadDetails";
import LeadTimeline from "./LeadTimeline";
import { StyledDrawer } from "./styles";

const DIALOG_NAME = "LeadDialog";

const LeadDrawer = ({ open, _setIsOpenDialog }) => {
  const isNew = open === true;
  const id = !isNew && open ? Number(open) : null;

  const { data: lead } = useGetLeadDetails(id);
  const { mutate: create, isSuccess: isCreateSuccess } = usePostLead();
  const { mutate: update, isSuccess: isUpdateSuccess } = usePutLead();

  const onClose = () => _setIsOpenDialog(DIALOG_NAME, false);

  const handleSubmit = (data) => {
    if (isNew) create(data);
    else update({ id, ...data });
  };

  // Auto-close on success
  useEffect(() => {
    if (isCreateSuccess || isUpdateSuccess) onClose();
  }, [isCreateSuccess, isUpdateSuccess]);

  return (
    <StyledDrawer anchor="right" open={!!open} onClose={onClose}>
      <HubHookFormWrapper defaultValues={isNew ? {} : { ...lead }} onSubmit={handleSubmit}>
        <Stack height="100%">
          <LeadBar onClose={onClose} isNew={isNew} />
          <Stack padding="20px" gap="20px" overflow="auto" flex={1}>
            <LeadDetails lead={lead} isNew={isNew} />
          </Stack>
          {!isNew && <LeadTimeline leadId={lead?.id} />}
        </Stack>
      </HubHookFormWrapper>
    </StyledDrawer>
  );
};

const mapStateToProps = (state) => ({
  open: getIsDialogOpen(state, DIALOG_NAME),
});
const mapDispatchToProps = { _setIsOpenDialog: setIsOpenDialog };

export const LeadDialog = connect(mapStateToProps, mapDispatchToProps)(LeadDrawer);
```

## Opening/Closing Dialogs

```javascript
import { setIsOpenDialog } from "../../redux/dialogs";

// Open for creating (open = true)
dispatch(setIsOpenDialog("LeadDialog", true));

// Open for editing (open = itemId)
dispatch(setIsOpenDialog("LeadDialog", leadId));

// Close (open = false)
dispatch(setIsOpenDialog("LeadDialog", false));
```

The `open` value is:
- `false` — dialog closed
- `true` — dialog open in "create" mode
- `number` — dialog open in "edit" mode with that entity ID

## Dialog Styling

```javascript
// styles.js
import { styled } from "@mui/material/styles";
import { Drawer } from "@mui/material";

export const StyledDrawer = styled(Drawer)({
  "& .MuiDrawer-paper": {
    width: "100%",
    maxWidth: "600px",
    overflow: "hidden",
  },
});
```

## HubDialogWrapper (Context-Based)

For simpler dialogs that don't need Redux, use `HubDialogWrapper` with context:

```jsx
import { HubDialogWrapper, useDialogContext } from "../../dialogs";

// Wrapper provides context
<HubDialogWrapper name="CreateDialog" customOnClose={() => refetch()}>
  <DialogContent />
</HubDialogWrapper>

// Inside children, access context
const { onClose, setOpen, open } = useDialogContext();
```

## Dialog Folder Structure

```
DialogName/
  DialogName.jsx          # Redux-connected drawer, form wrapper
  DialogNameBar.jsx       # Header with title, close button, submit button
  DialogNameDetails.jsx   # Form fields layout
  DialogNameTimeline.jsx  # Activity history (optional, for edit mode)
  helper.js               # Constants, field configs
  styles.js               # StyledDrawer, etc.
  index.js                # export { DialogName } from "./DialogName"
```

## Bar Component Pattern

```jsx
const LeadBar = ({ onClose, isNew }) => (
  <Stack direction="row" justifyContent="space-between" alignItems="center" p="15px">
    <Typography variant="h6">{isNew ? "New Lead" : "Edit Lead"}</Typography>
    <Stack direction="row" gap="10px">
      <HubButton type="submit" label="Save" />
      <IconButton onClick={onClose}>
        <HubIcon icon="X" size="16px" />
      </IconButton>
    </Stack>
  </Stack>
);
```

## Rules

- Dialogs render at the page level (alongside page content), not routed
- Use `anchor="right"` for right-side drawers (standard in this codebase)
- Max width: 600px for standard drawers
- Always wrap form content in `HubHookFormWrapper`
- Open/close state lives in Redux `dialogs` slice
- Auto-close on mutation success via `useEffect` on `isSuccess`
- Dialog name is a unique string constant (e.g., `"LeadDialog"`)
