# Setup Guide — Trap Door

A trap door is a floor tile that opens beneath the party's feet and drops them into a pit, dealing fall damage. It works in two modes: **automatic** — the trap fires the moment the party steps on it — and **lever-controlled** — a source opens the hatch only when the party is standing on it.

---

## Scene setup requirements

A trap door needs two things placed on the same grid cell:

1. A **floor tile** marked as Trap Door in the Dungeon Generator (`GridFlagExtended.cellType = TrapDoor`). This tells the grid that the cell is a trap door and the JSON export stores it accordingly.
2. A **TrapDoorTrap prefab** placed at the same world position as the floor tile. This is the component that actually detects the party and deals damage.

The `TrapMarker` component is required on the prefab root (it's enforced by `[RequireComponent]`). The Dungeon Generator uses it to assign a unique `trapId` and include the trap in the JSON export.

---

## Prefab hierarchy

```
TrapDoor                  ← root — TrapMarker, TrapDoorTrap
└── TrapMesh              ← trap geometry + Animator
```

---

## Step 1 — Mark the floor tile in the Dungeon Generator

In the Dungeon Generator's **Spawn Tools** panel, click the orange **Trap Door** button. This places a floor tile with `cellType = TrapDoor` at the origin — move it to the correct cell. Alternatively, select an existing floor flag and change its **Cell Type** to `TrapDoor` in the Inspector.

After placing all trap door tiles, click **Generate Unique IDs (all traps)** and then **Build Grid (Export JSON)** so the trap cell ends up in the grid data.

---

## Step 2 — Set up the Animator

Select **TrapMesh** and open its Animator Controller:

```
States
  Idle_Closed     ← default (floor tile looks normal)
  Opening         ← hatch swings open
  Idle_Open       ← pit is open
  Closing         ← hatch swings back (only needed when Permanent = false)

Triggers
  Open
  Close
```

**Transitions:**

- `Idle_Closed → Opening` — trigger `Open`, no Exit Time, Transition Duration 0.
- `Opening → Idle_Open` — Exit Time on, Transition Duration 0.
- `Idle_Open → Closing` — trigger `Close`, no Exit Time, Transition Duration 0. *(Only when Permanent = false)*
- `Closing → Idle_Closed` — Exit Time on, Transition Duration 0. *(Only when Permanent = false)*

If your trap has no animation, leave the Animator empty and leave **Trap Animator** unassigned — the trap still deals damage, just without a visual.

---

## Step 3 — Add TrapDoorTrap to the root

Add **TrapDoorTrap** to the `TrapDoor` root (it also requires `TrapMarker`, which is added automatically if missing).

### Auto-Trigger on Cell Enter

| Field | Default | Description |
|---|---|---|
| **Auto Trigger On Enter** | ✅ Enabled | The trap fires automatically when the party steps onto its cell. This is the classic "hole in the floor" mode. Disable for lever-controlled hatches. |
| **Require Party On Cell On External Trigger** | ✅ Enabled | For lever-controlled mode only (Auto Trigger disabled). The lever fires the trap only if the party is already standing on the cell. Disable to let a lever open the hatch regardless of where the party is standing. |
| **Trigger Delay** | 0.3 s | Seconds between the party entering the cell and the hatch opening. A slight delay (0.3–0.5 s) feels more natural — the hatch "gives way" under the player. 0 = immediate. |

### Behaviour

| Field | Default | Description |
|---|---|---|
| **Permanent** | ✅ Enabled | The hatch stays open after firing (the pit remains). Disable if you want the trap to reset and close after a delay. |
| **Close Delay** | 2 s | Seconds the hatch stays open before closing. Only used when Permanent is disabled. |
| **Trigger Once** | ✅ Enabled | After firing once, the trap ignores subsequent enters. Disable for a repeating trap. |

### Fall Damage

| Field | Default | Description |
|---|---|---|
| **Damage Preset** | InstantKill | Selects a preset damage value. **InstantKill** kills every party member immediately (game over). **Heavy** deals 50 damage. **Light** deals 20. **Custom** uses the value in Fall Damage below. |
| **Fall Damage** | 20 | Damage applied when Damage Preset is set to Custom. |

### Animator

| Field | Value |
|---|---|
| **Trap Animator** | Drag the `Animator` from `TrapMesh` |
| **Open Trigger** | `Open` |
| **Close Trigger** | `Close` |

### Events

Two `UnityEvent` fields let you hook in additional logic without code:

| Event | When it fires |
|---|---|
| **On Fall Triggered** | When the hatch opens and the party is on the cell. Use this to play a sound, trigger a camera shake, or run a cutscene. |
| **On Trap Closed** | When the hatch closes (only fires when Permanent = false). |

---

## Variant: lever-controlled hatch

Disable **Auto Trigger On Enter**. Add a lever (or any other `TriggerSource`) elsewhere in the dungeon and drag the `TrapDoorTrap` component into its **Targets** list.

When the lever is pulled, the trap checks whether the party is standing on the hatch cell. If they are, the hatch opens and fall damage is applied. If they are not, nothing happens — pulling the lever over an empty pit has no effect. This is controlled by **Require Party On Cell On External Trigger**.

Pulling the lever back (deactivating it) closes the hatch regardless of whether anyone is standing on it. This lets you use a lever as a toggle: open → party falls, close → hatch resets for the next victim.

---

## Variant: repeating trap

Set **Permanent = false**, **Trigger Once = false**, and choose a **Close Delay**. The sequence becomes: party enters → hatch opens → damage applied → wait Close Delay seconds → hatch closes → trap resets. The next time the party steps on the cell, the sequence repeats.

---

## Summary of components

| GameObject | Components |
|---|---|
| `TrapDoor` (root) | `TrapMarker`, `TrapDoorTrap` |
| `TrapMesh` (child) | `Animator` (Open/Close triggers, Idle_Closed / Idle_Open states) |

The floor tile on the same grid cell must have `cellType = TrapDoor` in its `GridFlagExtended` and be exported to the JSON before the trap functions correctly at runtime.

---

*CrawlerKIT — Mantis3de*
