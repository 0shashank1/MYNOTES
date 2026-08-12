

# My Network — Selection Persistence Across Store Reloads

  

## Summary

  

The **My Network** full-screen device grid (`WugDeviceGrid`) was losing the user's

row selection whenever the underlying store refreshed. A user would check several

devices (to run a bulk action, apply monitors, credentials, etc.), and a few

seconds later the checkboxes would silently clear on their own.

  

This document explains **why** it happened, **what** approach we took to fix it,

and **why** that approach was chosen over the alternatives.

  

> **Scope note.** My Network has **two independent selection pipelines** — the

> **full-screen grid** (backed by `pagedDeviceStore`) and the **map + docked

> split-grid** (backed by the NMD3 `d3Collection`). The reported bug and the

> primary fix below concern the **full-screen grid**. The **map + docked grid**

> is a separate architecture that is largely immune to the same bug by design; it

> is analysed in detail in the [Map view & docked grid](#map-view--docked-grid)

> section, including the one remaining edge-case gap and how to close it.

  

## Affected files

  

### Full-screen grid pipeline (the reported bug)

  

| File | Role |

| ---- | ---- |

| `WugDeviceGrid.js` (`deviceGrids/`) | The full-screen grid view. Hosts the `wugpagedgrid` bound to `pagedDeviceStore`. |

| `WugDeviceFullScreenController.js` | View controller. Owns selection tracking (`selectedRecords`), reload, and refresh logic. |

| `../grid/WugPagedGrid.js` | The grid class (`xtype: wugpagedgrid`), extends `MapGrid`. |

| `../grid/MapGrid.js` | Base grid. Declares the default `selModel` (`selType: 'mapgridmodel'`). |

| `../../../data/MapGridSelectionModel.js` | The checkbox selection model. Default `pruneRemoved: true`. |

| `../../../data/PagedDeviceStore.js` | The store. Fully reloads on a timer and on real-time events. |

| `../../../model/device/GeneralDeviceModel.js` | The record model. `idProperty: "DeviceId"`. |

  

### Map + docked-grid pipeline (analysed, separate architecture)

  

| File | Role |

| ---- | ---- |

| `../WugDeviceMap.js` | Map view. Embeds `xtype: 'wugdevicegrid'` as the docked south split-grid. |

| `../grid/WugDeviceGrid.js` (`grid/`) | The **docked** grid class (`xtype: wugdevicegrid`) — chained store over `d3Collection`. Different class from the full-screen `WugDeviceGrid`. |

| `../WugDeviceMapController.js` | Map view controller (extends `BaseMapController`). |

| `../BaseMapController.js` | Shared map selection logic: `restoreMapSelection`, `addMapSelections`, `setGridSelection`, node/grid sync. |

| `../grid/MapGridController.js` | Docked-grid controller. `applySourceFn` binds the chained store to `d3Collection.nodesDataCollection`. |

| `../MyNetwork/MyNetworkViewModel.js` | Defines both stores (`pagedDeviceStore`, `d3Collection`) and `reloadWugStore`. |

| `../BaseNetworkViewModel.js` | `reloadCollection()` → `d3Collection.load()` (merge path). |

| `NMD3/src/data/D3DataCollection.js` | The map collection. `mergeStoreNodeData` (in-place delta) vs `reload()` (clear+load). |

  

---

  

## The Issue

  

### Symptom

Selected device rows in the full-screen grid deselect themselves without user

interaction — typically within ~2 seconds of a device/map change, or on a fixed

timer.

  

### Root cause

The grid's store (`Wug.data.PagedDeviceStore`) is **fully reloaded** (destroy the

old records, fetch and build brand-new record instances) from several places:

  

1. **Periodic auto-refresh** — a `TaskManager` task started in the store

   constructor fires every **150 seconds** and calls `refreshPagedStore()`

   → `me.reload()`.

   ```js

   // PagedDeviceStore.js

   me.refreshStatusTask = {

       run: function () { /* ... */ me.refreshPagedStore(); },

       interval: 150000

   };

   ```

2. **Real-time device change** — `pagedDeviceChangeHandler` receives a

   "device modified/deleted" event and calls `refreshPagedStore()` (debounced 2s).

3. **Real-time map/group change** — `wugMapHandler` calls `refreshPagedStore()`

   for the current group.

4. **Filter / group / OR-filter changes** — the controller explicitly calls

   `reloadPagedDeviceGridStore()` → `store.load()`.

  

`store.reload()` / `store.load()` **removes every existing record and creates new

record objects** from the `/NmConsole/api/core/DeviceAggregates` response. Two

compounding factors then destroy the selection:

  

- **`pruneRemoved: true`** on `MapGridSelectionModel` — when a selected record is

  removed from the store (which happens to *all* records during a reload), the

  selection model prunes it from the selection **and fires a `selectionchange`

  with an empty selection**. The controller's `onGridRecordSelected` handler

  writes that empty selection straight into the ViewModel:

  ```js

  onGridRecordSelected: function (selModel) {

      vm.set("selectedRecords", selModel.getSelection()); // now []

  }

  ```

  So the reload not only clears the grid UI, it also erases our own record of

  what *was* selected — before anything can restore it.

  

- **New object identity** — even for devices that still exist after the reload,

  the reloaded rows are **different JavaScript object instances**. The selection

  model held references to the *old* instances, so it cannot re-associate them

  with the new rows on its own. Re-selection must be done by a **stable key**

  (`DeviceId`), not by object reference.

  

### Why it wasn't already handled

A `restoreGridSelection()` method already existed. It re-maps saved `DeviceId`s to

the newly loaded records and re-selects them. **But it was only wired to the

`onShow` event** (returning to the full screen). It was **never called on the

store's `load` event**, so the 150s timer and the real-time reloads cleared the

selection with nothing to put it back.

  

### Path that did NOT cause loss

The status-only real-time path (`pagedDeviceState`) updates records **in place**

via `record.set(...)` and only fires a custom `refreshView` event — it never

reloads the store, so it never lost selection. The loss came **exclusively** from

the full `reload()` / `load()` paths.

  

---

  

## The Fix

  

Two coordinated changes, following patterns already used elsewhere in the WUG

codebase (id-based reselection + `pruneRemoved: false`).

  

### 1. Stop the reload from pruning + erasing the selection

`WugDeviceGrid.js` — override the grid's `selModel` to set `pruneRemoved: false`

(while preserving the existing `selType` and `selectall` listener):

  

```js

selModel: {

    selType: 'mapgridmodel',

    pruneRemoved: false,

    listeners: {

        selectall: "onSelectAll"

    }

}

```

  

With `pruneRemoved: false`, a reload no longer strips the selection or fires the

spurious empty `selectionchange`. Our tracked `selectedRecords` survives the

reload so it can be re-mapped afterward.

  

### 2. Re-map the selection to the freshly loaded records after every reload

`WugDeviceFullScreenController.js` — call `restoreGridSelection()` from the store

`load` handler (`refreshPagedGridViewAfterLoad`), which runs after *every* reload

path, not just `onShow`:

  

```js

refreshPagedGridViewAfterLoad: function () {

    var me = this,

        overlayPicker = this.lookupReference("overlayFullScreenPicker");

  

    me.refreshPagedGridView();

    me.restoreGridSelection(); // re-map tracked selection to new records by DeviceId

    overlayPicker.setDisabledState(false);

}

```

  

`restoreGridSelection()` looks up each previously-selected `DeviceId` in the

reloaded store (`store.getById(...)`) and re-selects the matching new record

instances.

  

### 3. Replace (not append) the selection during restore

Because `pruneRemoved` is now `false`, stale record instances can linger in the

selection model after a reload. `restoreGridSelection()` therefore selects with

`keepExisting = false` so the stale entries are cleared and replaced by the

freshly loaded records:

  

```js

// was: grid.getSelectionModel().select(newRecords, true, true);

grid.getSelectionModel().select(newRecords, false, true);

//                                            ^ keepExisting = false (replace)

//                                                   ^ suppressEvent = true (avoid cyclic selectionchange)

```

  

This fix relies on `GeneralDeviceModel.idProperty === "DeviceId"`, which gives

each device a stable key across reloads.

  

---

  

## Why This Approach

  

We combined **two industry-standard techniques** for server-reloaded grids:

  

| Technique | Why we use it |

| --------- | ------------- |

| **`pruneRemoved: false`** | Built-in ExtJS config that keeps selected records through a store reload instead of dropping them (and the spurious empty `selectionchange`). Already used across WUG (`RemoteSitesLibrary`, `DeviceKeyGrid`, `PollersConfigLibraryEditor`, `DeviceRoleSelector`, …). |

| **Re-select by stable key (`DeviceId`) after `load`** | The canonical way to restore selection when a reload replaces record object instances. The `restoreGridSelection()` helper already existed; we simply wired it to the `load` event so it runs for *all* reload triggers. |

  

### Alternatives considered (and why not)

  

- **Delta / merge updates instead of full reload** — loading into the existing

  store so ExtJS merges by `idProperty` (persisted rows keep identity and

  selection). Cleanest long-term option and would also remove row flicker, **but**

  it changes the store/server contract and touches the periodic + three real-time

  reload paths — far higher risk and scope than the reported bug warranted.

- **Make the ViewModel the sole source of truth and re-apply on every render** —

  partially already true via `selectedRecords`; a full rewrite of the selection

  flow was unnecessary once the two targeted changes above closed the gap.

- **Disable / lengthen the 150s auto-refresh** — treats the symptom, not the

  cause; the real-time and filter reloads would still lose selection, and users

  would lose the live status updates the refresh provides.

  

### Correctness across scenarios

  

| Scenario | Behavior after fix |

| -------- | ------------------ |

| 150s periodic auto-refresh | Selection retained, re-mapped by `DeviceId`. |

| Real-time device modified/deleted | Selection retained; deleted devices simply drop out (no longer in store). |

| Real-time map/group change | Selection retained for devices still present. |

| Filter / OR-filter change | Filtered-out devices are not re-selected; still-present ones stay selected. |

| Group change | Existing `deselectAll()` clears `selectedRecords`; restore is a correct no-op — no cross-group carryover. |

| First load / empty selection | `restoreGridSelection()` is a no-op. |

  

---

  

## Scope & Notes

  

- Changes are **scoped to the My Network full-screen grid** (`WugDeviceGrid` +

  `WugDeviceFullScreenController`). The Discovery full-screen grid

  (`DiscoveryPagedGrid` / `DiscoveryDeviceFullScreenController`) shares the same

  architecture and its own `restoreGridSelection`; it was **not** modified as part

  of this change. Apply the same two-part fix there if the same symptom is

  observed.

- No new build/lint tooling was introduced. The modified files were validated with

  `node --check` (valid syntax). A full Sencha Cmd build/jshint pass should be run

  as part of normal CI.

  

## Verification (manual QA)

  

1. Open **My Network** and expand the full-screen device grid.

2. Select several devices via the checkbox column.

3. Wait for a refresh to occur — either the 150s timer, or trigger a real-time

   event (e.g. change a device / its state), or leave a filter applied.

4. **Expected:** the previously selected rows remain checked after the grid

   refreshes; selection count and any dependent bulk-action toolbar state are

   preserved.

5. Change the selected group in the device picker.

6. **Expected:** selection clears (correct — no carryover between groups).

  

---

  

## Map view & docked grid

  

My Network is really **two views over the same devices**: the full-screen grid

(above) and the **map** (which includes a docked/split grid along the bottom).

They are **separate selection pipelines** with separate stores, and it is

important to understand why the bug affects one but not the other.

  

### Two different `WugDeviceGrid` classes (do not confuse them)

  

| | Full-screen grid | Docked map grid |

| --- | --- | --- |

| Class | `Wug.view.device.deviceGrids.WugDeviceGrid` | `Wug.view.device.grid.WugDeviceGrid` |

| `xtype` | `wugdevicefullscreengrid` | `wugdevicegrid` |

| Grid inside | `wugpagedgrid` | itself |

| Store | **`pagedDeviceStore`** (flat records, `DeviceId`) | **chained store over `d3Collection.nodesDataCollection`** (nested `Wug.DeviceId` records) |

| Controller | `WugDeviceFullScreenController` | `WugDeviceMapController` → `BaseMapController` |

| Restore method | `restoreGridSelection()` | `restoreMapSelection()` |

  

Both share **one** ViewModel key — **`selectedRecords`** — so selecting a map

node checks the grid row and vice-versa. `restoreMapSelection()` re-applies

selection to **both** the map (`addMapSelections`) and the docked grid

(`setGridSelection`), so handling the map handles the docked grid for free.

  

### Data pipeline

  

```

NMD3 D3DataCollection (d3Collection)

        │  nodesDataCollection  (Ext.data.Store of D3 node records, id = Wug.DeviceId)

        ├──────────────► D3 map nodes  (rendered via NMD3 Select component)

        └──────────────► docked grid  (wugdevicegrid → CHAINED store over nodesDataCollection)

```

  

The docked grid binds its store in `MapGridController.applySourceFn`, which

returns `d3dc.nodesDataCollection` and refreshes on `mapdatachange` /

`mapdataupdate`.

  

### Why the map/docked grid does NOT have the full-screen grid's bug

  

The map collection has **two** update paths, and the common one preserves record

identity:

  

| Path | Mechanism | Node identity | Selection |

| --- | --- | --- | --- |

| **Merge / delta** (common) | `d3Collection.load()` → `mergeStoreNodeData()` classifies incoming nodes as **new / update / delete by ID**; existing nodes are updated **in place** | **Preserved** | **Preserved automatically** |

| **Clear-reload** (rare) | `d3Collection.reload()` = `clear()` (`removeAll`) + `load()` → every node record destroyed and rebuilt | **Destroyed** | **Lost** unless restored |

  

Because the periodic status updates and most refreshes go through the **merge**

path, the same record instances stay in `nodesDataCollection`. The selection

model still holds valid references, so the map + docked grid keep their selection

with **no extra code**. This is exactly the "delta/merge updates" industry-standard

approach (option #3 in [Alternatives considered](#alternatives-considered-and-why-not)),

and NMD3 already implements it. This is the reason the original bug was

**grid-only** — `pagedDeviceStore.reload()` is a destroy-and-recreate, whereas the

map merges.

  

### Every refresh trigger and its outcome

  

| Trigger | Path | Selection outcome |

| --- | --- | --- |

| Real-time device-state (status blips) | merge | kept ✔ |

| Group-card update / map reload event (`reloadCollection` → `d3dc.load()`) | merge | kept ✔ |

| Filter / recurse change | merge-load | kept ✔ |

| **Group change** (`onMapReloadEventAncClear`) | explicit `deselectAll` + clear `selectedRecords` | cleared — **correct**, no cross-group carryover ✔ |

| **Auto-layout toggle** (`reloadWugStore` → `d3Collection.reload()`) | **clear-reload** | **lost** ✖ |

| **Discovery "Start Monitoring" / export** (`bufferedReload` → `reloadWugStore`) | **clear-reload** | **lost** ✖ |

  

### The one remaining gap

  

Selection is lost on the two **clear-reload** triggers (auto-layout toggle,

discovery export), because `d3Collection.reload()` rebuilds all node records and

nothing re-selects afterward. Today `restoreMapSelection()` is wired **only** to

view-switch events:

  

- map `show` (`WugDeviceMap.js`),

- `onInitialResize`,

- `setLoadingState` — but only when the `skipMapSelection` flag is set (which is

  set when switching **from** the full screen back to the map).

  

It is **not** wired to run after a clear-reload, so those two structural refreshes

drop the selection.

  

> These are relatively rare, user-initiated structural changes (in the auto-layout

> case the nodes reposition anyway), which is why this gap is lower severity than

> the original grid bug. It is documented here so it can be closed deliberately.

  

### How to close the gap completely (recommended design)

  

The infrastructure already exists — reuse `restoreMapSelection()`, which re-maps

`selectedRecords` to the freshly loaded nodes by `Wug.DeviceId` and re-applies to

**both** the map and the docked grid. Mirror exactly what was done for the

full-screen grid (restore after load), using the **same flag pattern** the code

already uses for view-switches (`skipMapSelection`):

  

1. Before an explicit clear-reload (`reloadWugStore()` and any direct

   `d3Collection.reload()` path), set a flag on the ViewModel, e.g.

   `vm.set('restoreAfterReload', true)` (the current `selectedRecords` are already

   tracked in the VM, so no separate snapshot is required).

2. In the `mapdatachange` handler (`WugDeviceMapController.setLoadingState`), after

   load completes and `nodes.length !== 0`, if `restoreAfterReload` is set, clear it

   and call `this.restoreMapSelection()` — the same branch already used for

   `skipMapSelection`.

  

Why this design:

  

- **Reuses tested code** — `restoreMapSelection()` already handles the

  `Wug.DeviceId` re-mapping and updates both surfaces (map + docked grid).

- **Zero overhead on the common path** — guarded by the flag, so it does **not**

  fire on every merge/status blip; only after an actual clear-reload.

- **Consistent** with the existing `skipMapSelection` view-switch mechanism, so it

  fits the established pattern rather than inventing a new one.

- **Handles the docked grid automatically** — `restoreMapSelection` calls

  `setGridSelection`, so no separate grid handling is needed.

  

> **Status:** This map/docked-grid hardening is **analysed and specified here but

> not yet implemented**. The implemented fix in this document is the full-screen

> grid only. Implement the flag-based restore above if/when the auto-layout or

> discovery-export selection loss needs to be closed.

  

### Map/docked-grid QA (for when the gap is closed)

  

1. Open **My Network** in **map** view and select several devices (via node or the

   docked grid).

2. Let a real-time status refresh occur. **Expected today:** selection is retained

   (merge path).

3. Toggle **auto-layout**, or run a discovery "Start Monitoring" that completes.

   **Expected after the hardening:** selection is retained; **without** it,

   selection is lost (documented gap).

4. Change the selected group. **Expected:** selection clears (correct).