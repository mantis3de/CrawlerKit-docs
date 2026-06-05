# Trap Types — Classic Dungeon Traps

This guide covers 10 ready-to-use trap configurations built from `TrapDoorTrap` and `ProjectileTrap`. Each type describes what the player sees, what happens, the prefab structure, every Inspector field, and how to wire things together.

!!! note "Before you start"
    Read [Setup Guide — Pit & Trapdoor](setup-trapdoor.md) first — it explains the prefab hierarchy, TrapMarker, and the Animator step by step.

---

## Type 1 — Open Pit

**What happens:** The player sees a hole in the floor. They walk in and die instantly.

The simplest possible trap. No `TrapDoorTrap` component needed — the geometry and cell type do all the work.

**Setup:**

1. In the Dungeon Generator, mark the cell as **TrapDoor**.
2. Place the pit mesh (hole, walls on all sides, walls going down).
3. Add no components — done.

The party can enter the cell and falls immediately. Death is instant, no animation, no sound.

---

## Type 2 — Timed Trapdoor

**What happens:** The player steps onto what looks like a normal floor tile. After a moment the hatch opens beneath them and they fall.

The classic hidden trap — the player has no idea it's a trap until it's too late.

### Prefab hierarchy

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← closed hatch mesh (looks like a floor tile) + Animator
```

### Animator — states and transitions

Create an Animator Controller on `TrapMesh` with these states:

```
Idle_Closed    ← default state (right-click → Set as Layer Default State)
Opening        ← hatch swings open
Idle_Open      ← pit visible
```

In the **Parameters** tab, add one trigger:

| Name | Type |
|---|---|
| `Open` | Trigger |

Transitions:

| From | To | Condition | Has Exit Time |
|---|---|---|---|
| `Idle_Closed` | `Opening` | trigger `Open` | ❌ |
| `Opening` | `Idle_Open` | *(none — fires when clip ends)* | ✅ |

### Inspector — TrapDoorTrap

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.8–1.5 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |
| **Trap Animator** | Animator from TrapMesh |
| **Open Trigger** | `Open` |
| **Close Trigger** | *(leave empty — hatch never closes)* |

No lever needed. No external connections.

!!! tip
    The longer the **Trigger Delay**, the more sadistic the trap — the player hears the hatch creak and knows what's coming but can't step back in time.

---

## Type 3 — Lever-Controlled Trapdoor

**What happens:** The player is standing on a hatch. Someone (or the player themselves) pulls a lever — the hatch opens and they fall. Pulling the lever a second time closes the hatch again.

The classic trust trap — pull the lever and watch what happens.

### Prefab hierarchy

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← hatch mesh + Animator

[somewhere on a wall]
WallLever             ← WallLever (TriggerSource)
```

### Animator — states and transitions

Create an Animator Controller on `TrapMesh` with these states:

```
Idle_Closed    ← default state
Opening        ← hatch swings open
Idle_Open      ← pit visible
Closing        ← hatch swings shut
```

In the **Parameters** tab, add two triggers:

| Name | Type |
|---|---|
| `Open` | Trigger |
| `Close` | Trigger |

Transitions:

| From | To | Condition | Has Exit Time |
|---|---|---|---|
| `Idle_Closed` | `Opening` | trigger `Open` | ❌ |
| `Opening` | `Idle_Open` | *(none — fires when clip ends)* | ✅ |
| `Idle_Open` | `Closing` | trigger `Close` | ❌ |
| `Closing` | `Idle_Closed` | *(none — fires when clip ends)* | ✅ |

### Inspector — TrapDoorTrap

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ *(lever only fires if the party is standing on the hatch)* |
| **Trigger Delay** | 0.2 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trap Animator** | Animator from TrapMesh |
| **Open Trigger** | `Open` |
| **Close Trigger** | `Close` |

### Connecting the lever

1. Select the GameObject with `WallLever`.
2. In the Inspector, find the **Targets** list.
3. Click **+** and drag the GameObject with `TrapDoorTrap` into the slot.

First pull → lever activates → `OnTriggered` → hatch opens (`Open` fired).  
Second pull → lever deactivates → `OnReleased` → hatch closes (`Close` fired).

!!! tip
    Place the lever in the same room as the hatch — the player has to decide whether to stand on the hatch and pull, or investigate first.

---

## Type 4 — Lever Puzzle (Cycling Hatches)

**What happens:** Several hatches block the path. A single lever toggles their state — each pull opens and closes a different combination. The player must find the combination where all hatches on the path are closed and they can cross safely.

### Setup

Use one `WallLever` (toggle). Connect multiple `TrapDoorTrap` components as **Targets** on the lever.

Give each hatch a different **Trigger Delay** so they don't all open at once — they open in sequence, creating a shifting window of safe tiles:

| Hatch | Auto Trigger | Require Party | Trigger Delay | Permanent | Trigger Once |
|---|---|---|---|---|---|
| Hatch A | ❌ | ❌ | 0 s | ❌ | ❌ |
| Hatch B | ❌ | ❌ | 0.3 s | ❌ | ❌ |
| Hatch C | ❌ | ❌ | 0.6 s | ❌ | ❌ |

Lever activates → hatches open one by one. Lever deactivates → they close. The player toggles the lever and watches the rhythm to find the safe crossing window.

!!! note
    **Permanent = false** and **Trigger Once = false** on every hatch — otherwise they lock open on first trigger and won't reset.

### Animator

Each hatch needs the full state set from Type 3 (`Idle_Closed`, `Opening`, `Idle_Open`, `Closing`) and both triggers (`Open`, `Close`).

---

## Type 5 — Logic Gate Puzzle (Multi-Lever Lock)

**What happens:** Three levers control one or more hatches. The hatch opens only when all three levers are in the wrong position — the player must find the one safe combination.

### Setup

1. Create an empty GameObject and add a `TriggerLogicGate` component (type: **AND**).
2. Connect all three `WallLever` components as inputs to `TriggerLogicGate`.
3. Connect the `TriggerLogicGate` output to the `TrapDoorTrap`.

The hatch opens when all levers are active simultaneously. Every other combination is safe.

Invert the logic with a **NOT** or **NOR** gate — the hatch starts open and closes only when the player finds the correct combination.

### Inspector — TrapDoorTrap

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Permanent** | ❌ |
| **Trigger Once** | ❌ |

---

## Type 6 — Pressure Plate Trap

**What happens:** The player stands on a pressure plate — a hatch somewhere else opens. When the player steps off the plate, the hatch closes. The classic "hold the plate to cross" setup — but the plate opens the hatch under a second player.

### Setup

Wire `PressurePlate` → **Targets** → `TrapDoorTrap`.

### Inspector — TrapDoorTrap

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Trigger Delay** | 0 s |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |

### Animator

Full state set from Type 3 + both triggers `Open` and `Close`.

Player steps on plate → hatch opens. Next time anyone walks over the hatch → they fall.

---

## Type 7 — Repeating Pit

**What happens:** The hatch opens under the player, they fall and die. A few seconds later the hatch closes and resets — ready for the next victim.

Useful in corridors the player must travel through multiple times (backtracking, respawn points).

### Prefab hierarchy

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← Animator with full state set (Idle_Closed / Opening / Idle_Open / Closing)
```

### Animator

Identical to Type 3 — all four states and both triggers (`Open`, `Close`).

### Inspector — TrapDoorTrap

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ❌ |
| **Close Delay** | 3 s |
| **Trigger Once** | ❌ |
| **Trap Animator** | Animator from TrapMesh |
| **Open Trigger** | `Open` |
| **Close Trigger** | `Close` |

Sequence: party steps in → after 0.3 s hatch opens → party falls → after 3 s hatch closes → reset.

---

## Type 8 — Crossbow Corridor

**What happens:** The player enters a corridor, triggers a pressure plate, and an arrow fires from the wall.

The classic dart/arrow corridor — no hatch involved, just `ProjectileTrap` + `PressurePlate`.

### Setup

Wire `PressurePlate` → **Targets** → `ProjectileTrap`.

### Inspector — ProjectileTrap

| Field | Value |
|---|---|
| **Projectile Prefab** | arrow / bolt prefab |
| **Fire Direction** | perpendicular to the player's movement |
| **Damage** | 20–30 |
| **Aim Target** | Transform from `PartyVisuals` |

Place multiple `ProjectileTrap` components on both sides of the corridor with opposing **Fire Direction** values — arrows fire from left and right simultaneously.

---

## Type 9 — Pit + Crossbow Combo

**What happens:** The player steps onto a hatch cell. The hatch opens beneath them. At the same moment an arrow fires from the wall. Even if the hatch is lever-controlled and the player avoids standing on it, they still take the arrow.

`TrapDoorTrap` and `ProjectileTrap` triggered from the same source.

### Variant A — shared lever

One `WallLever` → **Targets**: both `TrapDoorTrap` and `ProjectileTrap` at the same time.

### Variant B — arrow fires on fall

Skip the lever. Use **On Fall Triggered** on `TrapDoorTrap` to call `ProjectileTrap.Trigger()` via UnityEvent. The arrow fires only after the hatch opens.

### Inspector — TrapDoorTrap

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **On Fall Triggered** | → `ProjectileTrap.Trigger()` |

### Inspector — ProjectileTrap

| Field | Value |
|---|---|
| **Fire Direction** | from the side wall |
| **Damage** | 25 |

---

## Type 10 — Gauntlet

**What happens:** The player must cross a corridor packed with hatches and arrows. Hatches open in sequence (each with a different Trigger Delay), arrows fire from a pressure plate at the corridor entrance. A safe path exists — the player has to find it.

Combines Type 4 (cycling hatches), Type 7 (repeating), and Type 8 (crossbow corridor).

### Setup

1. Lay out 4–6 `TrapDoor` cells along the corridor — each with a different **Trigger Delay** (0 s, 0.5 s, 1 s, 1.5 s…), **Permanent = false**, **Trigger Once = false**.
2. Connect one `WallLever` to all hatches — toggling it shifts which hatch is open at any given moment.
3. Place a `PressurePlate` at the corridor entrance wired to `ProjectileTrap` components on both sides.
4. The player reads the hatch rhythm first, then enters and sprints through the safety window.

### Inspector — each hatch

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Permanent** | ❌ |
| **Trigger Once** | ❌ |
| **Close Delay** | 2–3 s |
| **Trigger Delay** | different per hatch: 0, 0.5, 1.0, 1.5… |

### Animator — each hatch

Full state set from Type 3 (`Idle_Closed`, `Opening`, `Idle_Open`, `Closing`) + both triggers (`Open`, `Close`).

!!! tip
    Add **On Fall Triggered** → `TriggerPlaySound` on each hatch — the sound of the party falling, heard from the far end of the corridor, builds tension for the next attempt.

---

*CrawlerKIT — Mantis3de*
