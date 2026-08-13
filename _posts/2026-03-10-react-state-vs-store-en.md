---
layout: post
author: author1_en
tags: [posts-en]
title: "Don't Put React State Just Anywhere: When to Use a Store, and When Not To"
description: >
  The problems I ran into after dumping everything into a global store, and the criteria I now use to split state between local state, a global store, and server state.
---

While rebuilding an ITSM screen in React, I started out putting most of my state into a global store, on the reasoning that "if it's in the store, I can pull it out from anywhere, so it's convenient." As the number of screens grew, that became a real problem.

## 1. What Went Wrong

- **Unnecessary re-renders**: even values scoped to a single component — like a search box's input value — ended up in the store, so every keystroke re-rendered the entire screen subscribed to that store. As screens grew larger, typing visibly started to lag.
- **Hard to trace data flow**: figuring out when and why a value changed meant digging through every store action. Looking at a component alone, you couldn't see "this value starts here and flows there."
- **Freshness problems with server-fetched data**: I stored list-query API responses directly in the store and pulled them from wherever I needed them, but if another screen updated the data and forgot to refresh the store, different screens ended up showing different values for the same thing. The entire responsibility for guaranteeing "is this value actually current" was left on the developer's shoulders.

## 2. Thinking of State in Three Categories

| Category | Examples | Where it belongs |
|---|---|---|
| **Local UI state** | input values, modal open/closed, hover state on a specific row | `useState`/`useReducer` (scoped to that component) |
| **Shared UI state** | logged-in user info, dark mode toggle, filter conditions multiple screens need to see | global store (Redux/Zustand/Context) |
| **Server state** | list-query results, detail-query results — anything the server owns as the source of truth | a server-state caching library like React Query |

When you don't distinguish between these three and dump everything into a single "global store" bucket, the store ends up doing double duty as both a UI state manager and a server-data cache, and the problems above follow naturally.

## 3. Decision Criteria

The order of questions I now ask myself when deciding where a new piece of state should live:

1. **Is this value only used by one component (or its subtree)?** → `useState` is enough. There's no reason to lift it into the store.
2. **Is the source of truth for this value a server response?** → don't copy it into the store and manage it there; hand it off to a server-state caching library. The store should only hold UI state about how to *display* that data, not the server data itself.
3. **Do multiple, unrelated screens genuinely need to share the same value in real time?** → only then does it belong in the global store.

Criterion 3 turned out to matter the most. The habit of pre-emptively putting something in the store "just in case another screen needs it later" was where the trouble started. Keeping things component-local until they were actually needed elsewhere, and only lifting them into the store once multiple screens genuinely needed to share them, ended up producing less code overall.

## 4. Grid Event Callbacks Referencing Stale Store Values

Getting the store placement right wasn't the end of it. I got tripped up again in the logic that refetches a list after a save and restores the just-edited row as the selected row.

```js
// The problematic code
const gridEvents = useMemo(
  () => ({
    onCurrentRowChanged(_grid, _oldRow, newRow) {
      if (newRow >= 0 && rowList[newRow]) {
        setSelectedRow(rowList[newRow]);
      } else {
        setSelectedRow(null); // this branch kept firing unintentionally
      }
    },
  }),
  [rowList, setSelectedRow]
);

// Refetch after save, then restore the just-saved row as selected
fetchList(() => {
  const idx = list.findIndex(row => row.id === savedId);
  if (idx >= 0) {
    setTimeout(() => {
      gridRef.current?.vw?.setCurrent({ dataRow: idx }); // workaround: just wait 100ms and hope
    }, 100);
  }
});
```

`gridEvents` is a callback that only gets recreated when `rowList` changes. But when I refetch the list, the store updates, and then immediately call `setCurrent` on the grid to restore the selection, the `onCurrentRowChanged` the grid fires at that exact moment could still be **a callback closing over the old, not-yet-replaced `rowList`.** The `rowList` inside that stale closure reflected the pre-refetch state, so the index was off or the value was missing, sending it down the `else` branch — overwriting what should have been the freshly selected row's detail with `null` or a leftover previous value. I tried papering over it with `setTimeout(..., 100)`, but that's really just hoping the re-render finishes in that window, and it kept reproducing on slower environments.

The root cause was a possible gap in timing between "when the event fires" and "when React swaps in the callback that's supposed to handle it with the latest state." So instead of relying on a value captured in a closure inside the callback, I changed it to read the store's value directly, at the exact moment the callback runs.

```js
// Fixed — read the store's current value directly instead of relying on a closure
const gridEvents = {
  onCurrentRowChanged(_grid, _oldRow, newRow) {
    const rowList = useGridStore.getState().rowList; // always the latest value
    if (newRow >= 0 && rowList[newRow]) {
      setSelectedRow(rowList[newRow]);
    } else if (newRow < 0) {
      setSelectedRow(null);
    }
  },
};

fetchList(() => {
  const { rowList } = useGridStore.getState();
  const idx = rowList.findIndex(row => row.id === savedId);
  if (idx >= 0) {
    gridRef.current?.vw?.setCurrent({ dataRow: idx }); // safe to call immediately, no setTimeout needed
  }
});
```

Zustand's `getState()` reads a snapshot of the current value immediately, with no subscription involved. It always reflects the latest value regardless of what happened to be captured in a closure, which meant the whole exercise of trying to time things with `setTimeout` became unnecessary. This case made it clear to me that "destructuring a store value during component render" and "reading a value at the moment an event callback fires" are genuinely different problems.

## 5. Wrap-Up

I came away with a clear sense that a store should be treated not as "a warehouse for stashing state," but as "a place you deliberately promote only the state that multiple screens genuinely need to share." Keeping values local when local is enough, handing server-sourced data off to a dedicated server-state tool, and leaving only truly global state in the store noticeably cut down on both re-render issues and data-consistency issues. And for spots like event callbacks — places that can fire asynchronously, outside React's normal render cycle — building the habit of reading the store's value directly at that moment, rather than trusting a closure, fundamentally cut down on timing bugs.
