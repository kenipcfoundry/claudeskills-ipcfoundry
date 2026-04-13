# Frontend Project Standards

This project follows company coding conventions. Always reference and apply the
appropriate skill for every task, regardless of how the request is phrased.

## Skill Usage Rules

When building or modifying ANY of the following, always load and follow the
corresponding skill before writing any code:

- Any form, input, select, date picker, or switch
  → use `/frontend-conventions:frontend-forms`

- Any data table, list with columns, or paginated data view
  → use `/frontend-conventions:frontend-tables`

- Any dialog, drawer, modal, or slide-out panel
  → use `/frontend-conventions:frontend-dialogs`

- Any styling, colors, typography, spacing, or styled components
  → use `/frontend-conventions:frontend-styling`

- Any API call, data fetching, mutation, or cache invalidation
  → use `/frontend-conventions:frontend-react-query`

- Any Redux state, Context provider, or SSE real-time update
  → use `/frontend-conventions:frontend-state-management`

- Any drag-and-drop, kanban board, or reorderable list
  → use `/frontend-conventions:frontend-drag-drop`

- Any icon, button, navigation tab, or page layout
  → use `/frontend-conventions:frontend-icons-nav`

- Creating a new page, component, container, or folder structure
  → use `/frontend-conventions:frontend-project-structure`

## General Rules

- Always apply these conventions even if the user just says "build this",
  "add a feature", or "fix this bug"
- Never use raw CSS files — always use MUI styled() or sx prop
- Never store API data in Redux — use React Query
- Always use HubHookFormWrapper for any form
- Always use HubTableTemplate for any data table
- Always use Redux dialogs slice for dialog open/close state
- Import components from the barrel: `../../components`
