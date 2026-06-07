# Setup Guide — Pressure Plate Door

This guide covers two variants of a pressure plate door:

- **Closes on release** — the door opens when the player steps on the plate and closes when they step off.
- **Stays open** — the door opens when the player steps on the plate and stays open permanently; stepping off has no effect.

---

## What you need before you start

- A door prefab with a mesh and an `Animator`.
- A pressure plate prefab — a flat floor tile with a mesh, an `Animator` and a `Collider`.

---

## Prefab hierarchy

```
Door                      ← door root
├── DoorMesh              ← door geometry + Animator

PressurePlate             ← plate root (placed on a floor cell)
└── Model                 ← plate geometry + Animator
```

The plate and door are separate GameObjects. The plate holds a reference to the door target in its **Targets** list.

---

## Step 1 — Set up the door Animator

Select **DoorMesh** and open its Animator Controller:

```
States
  Idle_Closed     ← default
  Opening
  Idle_Opened
  Closing

Triggers
  Open
  Close
```

**Transitions:**

- `Idle_Closed → Opening` — trigger `Open`, no Exit Time, Transition Duration 0.
- `Opening → Idle_Opened` — Exit Time on, Transition Duration 0.
- `Idle_Opened → Closing` — trigger `Close`, no Exit Time, Transition Duration 0.
- `Closing → Idle_Closed` — Exit Time on, Transition Duration 0.

---

## Step 2 — Set up the plate Animator

Select the plate's **Model** child and open its Animator Controller:

```
States
  Idle_Up         ← default (plate unpressed)
  Idle_Down       ← plate pressed

Triggers
  Press
  Release
```

**Transitions:**

- `Idle_Up → Idle_Down` — trigger `Press`, no Exit Time, Transition Duration 0.
- `Idle_Down → Idle_Up` — trigger `Release`, no Exit Time, Transition Duration 0.

The plate snaps immediately between up and down — no separate press/release animations are required, though you can add them if you have them.

---

## Step 3 — Add components to the door

Add **EdgeBlockerMarker** and **TriggerOpenDoor** to the `Door` root. The fields are the same for both variants:

| Field | Value |
|---|---|
| **Blocker Id** | Unique ID, e.g. `door_trap_01` |
| **Door Animator** | Drag the `Animator` from `DoorMesh` |
| **Open Anim Trigger** | `Open` |
| **Close Anim Trigger** | `Close` |
| **Opened State Name** | `Idle_Opened` |
| **Closed State Name** | `Idle_Closed` |

The two variants differ only in **Close On Release**:

| Variant | Close On Release |
|---|---|
| Closes when stepped off | ✅ Enabled |
| Stays open permanently | ❌ Disabled |

Leave **Skip Close Guard** disabled in both cases.

---

## Step 4 — Add PressurePlate to the plate root

Select the `PressurePlate` root and add **PressurePlate**:

| Field | Description |
|---|---|
| **Source Id** | Unique string, e.g. `plate_trap_01`. Required if you want the plate state saved. |
| **React To Party** | ✅ Enabled — the plate responds to the player standing on it. |
| **React To Enemies** | Optional. Enable if you want enemies walking onto the plate to also trigger it. |
| **Latching** | Leave disabled. Latching means the plate stays activated even after the weight is removed — useful for one-way triggers, but not for this use case. |
| **Required Weight** | Leave at 0 for standard presence-only detection. Set a positive value (kg) if you want the plate to require a certain weight of items dropped on it before it activates. |
| **Plate Animator** | Drag the `Animator` from the `Model` child. |
| **Press Anim Trigger** | `Press` |
| **Release Anim Trigger** | `Release` |
| **Targets** | Drag the `TriggerOpenDoor` component from the door root. |

!!! note "Interact From Adjacent Cell"
    `PressurePlate` is a floor trigger — it activates automatically when the player steps onto its cell. It does not use mouse clicks or the **Interact From Adjacent Cell** setting.

---

## How the close guard works with pressure plates

When **Close On Release** is enabled and the player steps off the plate, the door does not close immediately. It first checks that neither the player nor any living enemy is standing in the doorway. If the doorway is occupied the door waits, polling every 0.1 seconds, and closes as soon as both cells are clear.

This means the player can step off the plate and stand in the doorway — the door stays open until they move through.

---

## Variant: stays open permanently

With **Close On Release** disabled, `TriggerOpenDoor` ignores the `Released` event entirely. The door opens the first time the plate is stepped on and never closes again regardless of what happens to the plate.

If you want this door to be openable only once (no re-closing from any source), this is the correct setup. If you want the door to be closable via a lever elsewhere in the dungeon, wire a second `TriggerOpenDoor` target on that lever instead.

---

## Step 5 — Export

In the Dungeon Generator, click **Generate Unique IDs (all doors)** then **Build Grid (Export JSON)**.

---

## Testing

Press Play. Walk onto the plate cell. The plate animates down and the door opens. For the **closes on release** variant, walk off the plate — the door waits for the doorway to clear, then closes. For the **stays open** variant, the door remains open regardless of where you stand.

If the plate activates but the door doesn't open, check that the `TriggerOpenDoor` component appears in the **Targets** list on `PressurePlate` and that `TriggerOpenDoor.BlockerId` matches `EdgeBlockerMarker.blockerId`.

---

## Summary of components

| GameObject | Components |
|---|---|
| `Door` (root) | `EdgeBlockerMarker`, `TriggerOpenDoor` |
| `DoorMesh` (child) | `Animator` (Open/Close triggers, four states) |
| `PressurePlate` (root) | `PressurePlate` |
| `Model` (child) | `Animator` (Press/Release triggers, two states) |

---

*CrawlerKIT — Mantis3de*
