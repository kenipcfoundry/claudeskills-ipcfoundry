---
name: frontend-icons-nav
description: HubIcon component, HubButton, react-feather icon library, SideNavBarV2 page tabs, TopNavBar, and navigation patterns for the frontend repository.
---

# Frontend Icons & Navigation

## HubIcon

Universal icon component wrapping react-feather icons + custom SVGs in MUI `SvgIcon`:

```jsx
import { HubIcon } from "../../components";

<HubIcon icon="Edit" size="12px" />
<HubIcon icon="ChevronDown" size="16px" sx={{ color: "hubColors.grey" }} />
<HubIcon icon="AlertTriangle" size="20px" />
```

**Props:**
- `icon` — icon name (PascalCase string)
- `size` — pixel string (default: `"16px"`)
- `sx` — MUI sx prop for styling
- All other props spread to `SvgIcon`

## Available Icons

**react-feather icons** (PascalCase name): `Edit`, `Trash`, `Trash2`, `Plus`, `Minus`, `ChevronDown`, `ChevronRight`, `ChevronLeft`, `ChevronUp`, `Search`, `FileText`, `File`, `X`, `Check`, `AlertTriangle`, `AlertCircle`, `Info`, `Settings`, `User`, `Users`, `Mail`, `Phone`, `Calendar`, `Clock`, `Filter`, `Download`, `Upload`, `ExternalLink`, `Copy`, `Eye`, `EyeOff`, `Lock`, `Unlock`, `MoreVertical`, `MoreHorizontal`, `Printer`, `RefreshCw`, `Save`, `Send`, `Star`, `Truck`, `Package`, `MapPin`, `ArrowLeft`, `ArrowRight`, `Menu`, `Bell`, `Home`, `Layers`, `Grid`, `List`, `Move`, `Maximize`, `Minimize`

**Custom SVG icons**: `Fire`, `Hand`, `Shop`, `Tag`, `Stove`, `Check` (custom), `Barcode`, `FileSolid`, `Priority`

## HubButton

Custom button with optional icon:

```jsx
import { HubButton } from "../../components";

// Text button with icon (ghost until hovered)
<HubButton variant="text" icon="Edit" iconProps={{ size: "12px" }} label="Edit" size="xsmall" />

// Primary button
<HubButton label="Save" type="submit" />

// Icon-only button
<HubButton variant="text" icon="Plus" size="small" />
```

## Page Tabs (SideNavBarV2)

Vertical tab navigation for pages:

```jsx
import { SideNavBarV2 } from "../../components";

const TABS = [
  { id: 1, label: "Quotes", icon: "FileText" },
  { id: 2, label: "Leads", icon: "Users" },
];

<SideNavBarV2 tabs={TABS} value={tab} setValue={setTab} />
```

**Props:**
- `tabs` — array of `{ id, label, icon }` (icon is a HubIcon name)
- `value` — currently selected tab ID
- `setValue` — callback `(id) => {}`
- `EndComponent` — optional component rendered below tabs

## Page Layout Pattern

```jsx
const SalesPage = () => {
  const [tab, setTab] = useState(1);
  const [viewMode, setViewMode] = useState("BOARD");

  return (
    <PageContent>
      <Stack direction="row" height="100%">
        <SideNavBarV2 tabs={TABS} value={tab} setValue={setTab} />
        <Stack flex={1} overflow="hidden">
          <SalesBar viewMode={viewMode} setViewMode={setViewMode} />
          {getView(tab, viewMode)}
        </Stack>
      </Stack>
      <LeadDialog />
      <QuoteDialog />
    </PageContent>
  );
};
```

## TopNavBar

Always rendered for authenticated routes. Contains:
1. Logo + menu buttons (filtered by user permissions)
2. CreateButton (context-sensitive create actions)
3. SearchNavBar (if user has search permission)
4. NotificationBell
5. UserProfileContextMenu

Menu items are filtered by `getUserMenuItems(permissions)` from `constants/menu.js`.

## Navigation Helpers

```jsx
// In table cells via keys array
{ value: "workOrder", variant: "blueBold", navigate: true, route: "works", id: "workOrder" }
// Clicking navigates to /works/{item.workOrder}

// Programmatic navigation
const navigate = useNavigate();
navigate(`/works/${workOrderNumber}`);

// URL params
const { id } = useParams();

// Search params (for tabs/filters)
const [searchParams, setSearchParams] = useSearchParams();
setSearchParams({ tab: 1, view: "BOARD" }, { replace: true });
```
