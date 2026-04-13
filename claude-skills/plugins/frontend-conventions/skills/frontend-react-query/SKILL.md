---
name: frontend-react-query
description: React Query data fetching patterns for the frontend repository. Covers query/mutation function extraction, useQuery/useMutation hooks, cache invalidation, snackbar feedback, and query key conventions.
---

# Frontend React Query Hooks

One file per API resource in `hooks/react-query/`. Follow this exact pattern.

## Standard Hook File

```javascript
import { useMutation, useQuery, useQueryClient } from "react-query";
import qs from "qs-lite";

import api from "../../utils/api";
import { useHubSnackBar } from "../useSnackBar";
import { invalidate } from "../../helpers/clientHelper";

const RESOURCE = "/resource";

// ── Queries ──

// 1. Extract query function as named module-level function
const getItems = ({ queryKey }) => {
  const queryString = qs.stringify(queryKey[1]);
  return api.get(`${RESOURCE}?${queryString}`);
};

// 2. Export hook that wraps it
export const useGetItems = ({ location }) => {
  return useQuery([RESOURCE, { location }], getItems, {
    enabled: !!location,
  });
};

// Detail query — extract ID from queryKey
const getItemDetails = ({ queryKey }) => api.get(`${RESOURCE}/${queryKey[1]}`);
export const useGetItemDetails = (id) =>
  useQuery([RESOURCE, id], getItemDetails, { enabled: !!id });

// History/sub-resource query
const getItemHistory = ({ queryKey }) =>
  api.get(`${RESOURCE}/${queryKey[1]}/history`);
export const useGetItemHistory = (id) =>
  useQuery([RESOURCE, id, "history"], getItemHistory, { enabled: !!id });

// ── Mutations ──

const postItem = (data) => api.post(RESOURCE, data);
export const usePostItem = () => {
  const { addSnackbarSuccess } = useHubSnackBar();
  const queryClient = useQueryClient();

  return useMutation(postItem, {
    onSuccess: () => {
      invalidate(queryClient, RESOURCE);
      addSnackbarSuccess({ message: "Successfully Created Item" });
    },
  });
};

const putItem = ({ id, ...data }) => api.put(`${RESOURCE}/${id}`, data);
export const usePutItem = () => {
  const { addSnackbarSuccess } = useHubSnackBar();
  const queryClient = useQueryClient();

  return useMutation(putItem, {
    onSuccess: () => {
      invalidate(queryClient, RESOURCE);
      addSnackbarSuccess({ message: "Successfully Updated Item" });
    },
  });
};

const deleteItem = (id) => api.delete(`${RESOURCE}/${id}`);
export const useDeleteItem = () => {
  const { addSnackbarSuccess } = useHubSnackBar();
  const queryClient = useQueryClient();

  return useMutation(deleteItem, {
    onSuccess: () => {
      invalidate(queryClient, RESOURCE);
      addSnackbarSuccess({ message: "Successfully Deleted Item" });
    },
  });
};
```

## Optimistic Updates (Kanban Drag)

For drag-and-drop stage changes, use `onMutate` for instant UI:

```javascript
const putStage = ({ id, stage }) =>
  api.put(`${RESOURCE}/${id}/stage`, { stage });

export const usePutStage = () => {
  const { addSnackbarError } = useHubSnackBar();
  const queryClient = useQueryClient();

  return useMutation(putStage, {
    onMutate: async ({ id, stage }) => {
      await queryClient.cancelQueries(RESOURCE);
      const previous = queryClient.getQueryData(RESOURCE);

      if (Array.isArray(previous)) {
        const updated = previous.map((col) => ({
          ...col,
          elements: col.elements.filter((e) => String(e.id) !== String(id)),
        }));
        const source = previous.find((col) =>
          col.elements.some((e) => String(e.id) === String(id)),
        );
        const moved = source?.elements.find((e) => String(e.id) === String(id));
        const target = updated.find((col) => String(col.id) === String(stage));
        if (target && moved) target.elements.push({ ...moved, stage });
        queryClient.setQueryData(RESOURCE, updated);
      }

      return { previous };
    },
    onError: (_err, _vars, ctx) => {
      ctx?.previous && queryClient.setQueryData(RESOURCE, ctx.previous);
      addSnackbarError({ message: "Failed to move item" });
    },
    onSettled: () => invalidate(queryClient, RESOURCE),
  });
};
```

## Cache Invalidation

```javascript
// helpers/clientHelper.js
export const invalidate = (client, key) =>
  client.invalidateQueries({
    predicate: (query) => query.queryKey[0] === key,
    exact: false,
  });
```

Invalidates ALL queries whose first key segment matches. So `invalidate(client, "/schedule")` clears `["/schedule", {...}]`, `["/schedule/throughput", {...}]`, etc.

## Query Key Convention

```javascript
// List: [endpoint, params]
["/items", { location: 1, status: "active" }]

// Detail: [endpoint, id]
["/items", 42]

// Sub-resource: [endpoint, id, sub]
["/items", 42, "history"]

// Nested with customer: [endpoint, "customer", customerId, params]
["/items", "customer", 15, { mode: "complete" }]
```

## Rules

- **Always extract** the API call function as a **named module-level function** above the hook (never inline `() => api.get(...)`)
- Query key convention: `[ENDPOINT, params]` or `[ENDPOINT, id]`
- Use `invalidate(queryClient, KEY)` to clear all queries matching that endpoint prefix
- Use `useHubSnackBar()` for success/error feedback in every mutation
- Use `{ enabled: !!condition }` for conditional fetching
- For `useQuery` functions receiving `{ queryKey }`, destructure params from `queryKey[1]`, `queryKey[2]`, etc.
- One file per API resource: `useItemData.js`, `useScheduleData.js`, etc.
- Endpoint constant at top of file: `const RESOURCE = "/items"`
