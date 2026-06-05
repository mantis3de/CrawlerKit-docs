# Setup Guide — Pit & Trapdoor

Every hole in the floor — a plain open pit or an animated trapdoor — uses the same **TrapDoor** cell type and the same pit mesh (dark void, walls on all sides, walls going down). The difference is whether you attach a `TrapDoorTrap` component:

| Setup | Cell type | Component | Behaviour |
|---|---|---|---|
| Plain pit | TrapDoor | none | Party walks in, falls instantly |
| Trap pit | TrapDoor | TrapDoorTrap | Full mechanics: levers, delays, reset, custom damage |

---

## Scene setup requirements

1. A **floor cell** marked as **TrapDoor** in the Dungeon Generator (`GridFlagExtended.cellType = TrapDoor`).
2. A **pit mesh prefab** placed at the same world position — the pit mesh looks the same in both cases: a hole with walls on all sides and walls going down.
3. *(Optional)* A `TrapDoorTrap` component on the prefab root if you want trap mechanics.

---

## Step 1 — Mark the cell in the Dungeon Generator

In the Dungeon Generator's **Spawn Tools** panel, click the orange **Trap Door** button. This places a floor cell with `cellType = TrapDoor` at the origin — move it to the correct position. Alternatively, select an existing floor flag and change its **Cell Type** to `TrapDoor` in the Inspector.

After placing all cells, click **Generate Unique IDs (all traps)** and then **Build Grid (Export JSON)**.

!!! note "No more Pit cell type"
    The old **Pit** cell type (impassable, decorative) is no longer used. All holes in the floor use **TrapDoor**. A plain pit without a `TrapDoorTrap` component behaves identically to the old Pit — the party can enter the cell and falls — while still allowing you to add trap mechanics later simply by attaching the component.

---

## Step 2 — Set up the prefab

### Plain pit (no component)

Place the pit mesh at the cell. No `TrapMarker` or `TrapDoorTrap` needed. The party walks in and falls immediately. Done.

### Trap pit (with component)

```
TrapDoor                  ← root — TrapMarker, TrapDoorTrap
└── TrapMesh              ← pit geometry + Animator (optional)
```

Add `TrapMarker` and `TrapDoorTrap` to the root. `TrapMarker` is required for the Save System — it stores the `trapId` and registers the trap in the JSON export.

---

## Step 3 — Configure TrapDoorTrap

### Auto-Trigger on Cell Enter

| Field | Default | Description |
|---|---|---|
| **Auto Trigger On Enter** | ✅ Enabled | Fires automatically when the party steps onto the cell. Disable for lever-controlled traps. |
| **Require Party On Cell On External Trigger** | ✅ Enabled | For lever-controlled mode. The lever only works if the party is already standing on the cell. |
| **Trigger Delay** | 0 s | Seconds between entering the cell and applying damage. Use 0.2–0.4 s if you have an opening animation. |

### Behaviour

| Field | Default | Description |
|---|---|---|
| **Permanent** | ✅ Enabled | The hole stays open after firing. Disable if the trap should close and reset. |
| **Close Delay** | 2 s | Seconds before the trap resets. Only used when Permanent is disabled. |
| **Trigger Once** | ✅ Enabled | Ignore subsequent enters after the first trigger. Disable for a repeating trap. |

### Fall Damage

| Field | Default | Description |
|---|---|---|
| **Damage Preset** | InstantKill | `InstantKill` kills everyone immediately. `Heavy` deals 50. `Light` deals 20. `Custom` uses the value below. |
| **Fall Damage** | 20 | Damage value when Damage Preset is Custom. |

### Animator (optional)

Only needed for an animated hatch that visually opens. Leave all animator fields empty for a pit that is always open.

| Field | Value |
|---|---|
| **Trap Animator** | Animator from `TrapMesh` |
| **Open Trigger** | `Open` |
| **Close Trigger** | `Close` |

If you assign an animator, set up these states and transitions:

```
States
  Idle_Closed     ← default (hatch looks closed, floor visible)
  Opening         ← hatch swings open
  Idle_Open       ← pit is visible
  Closing         ← hatch swings back (only when Permanent = false)
```

Transitions: `Idle_Closed → Opening` on trigger `Open` (no Exit Time). `Opening → Idle_Open` on Exit Time. Same pattern for Close.

### Events

| Event | When it fires |
|---|---|
| **On Fall Triggered** | Just before damage is applied. Wire a sound, camera shake, or cutscene. |
| **On Trap Closed** | When the hatch closes (only when Permanent is disabled). |

---

## Variant: lever-controlled trapdoor

Disable **Auto Trigger On Enter**. Place a lever (or any `TriggerSource`) in the scene and drag the `TrapDoorTrap` component into its **Targets** list. The lever opens the floor only when the party is standing on the cell. **Require Party On Cell On External Trigger** controls this — disable it to let the lever open the floor regardless of where the party is.

---

## Variant: repeating trap

Set **Permanent = false**, **Trigger Once = false**, and a **Close Delay**. The sequence: party enters → falls → wait Close Delay → trap resets → next victim.

---

## Summary

| GameObject | Components |
|---|---|
| `TrapDoor` (root) | `TrapMarker` *(required)*, `TrapDoorTrap` *(optional)* |
| `TrapMesh` (child) | `Animator` *(optional — only for animated hatch)* |

The cell on the same grid position must have `cellType = TrapDoor` and be exported to the JSON.

---

*CrawlerKIT — Mantis3de*
