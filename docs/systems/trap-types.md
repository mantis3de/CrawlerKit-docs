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

The party walks in and falls immediately. No animation, no sound.

---

## Type 2 — Timed Trapdoor

**What happens:** The player steps onto what looks like a normal floor tile. After a moment the hatch opens beneath them and they fall. The hatch can also be closed again by a lever.

The classic hidden trap — the player has no idea it's a trap until it's too late.

### Prefab hierarchy

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← closed hatch mesh (looks like a floor tile) + Animator
```

### Animator — states

Open the Animator window (**Window → Animation → Animator**). Create four states:

```
Idle_Closed    ← default state — hatch looks like a normal floor tile
Opening        ← hatch swings open animation
Idle_Open      ← hatch is open, pit is visible
Closing        ← hatch swings shut animation
```

Set **Idle_Closed** as the default: right-click it → **Set as Layer Default State**. It will turn orange.

### Animator — parameters

In the **Parameters** tab (top-left of the Animator window), click the **+** button and add two Triggers:

| Name | Type |
|---|---|
| `Open` | Trigger |
| `Close` | Trigger |

### Animator — transitions

To create a transition: **right-click** the source state → **Make Transition** → click the destination state. A white arrow appears. Click the arrow to select it and configure it in the Inspector.

---

**Transition 1: `Idle_Closed` → `Opening`**

This fires when the trap opens (player steps on it or lever pulls it).

In the Inspector for this transition:

| Setting | Value |
|---|---|
| Has Exit Time | ❌ off |
| Transition Duration | 0 |
| Transition Offset | 0 |

In the **Conditions** list, click **+** and set:

| Parameter | Value |
|---|---|
| `Open` | *(just select the trigger — no value needed)* |

---

**Transition 2: `Opening` → `Idle_Open`**

This fires automatically when the opening animation finishes playing.

In the Inspector for this transition:

| Setting | Value |
|---|---|
| Has Exit Time | ✅ on |
| Exit Time | 1.0 *(full clip plays before transitioning)* |
| Transition Duration | 0 |

Leave the **Conditions** list empty.

---

**Transition 3: `Idle_Open` → `Closing`**

This fires when a lever closes the hatch.

In the Inspector for this transition:

| Setting | Value |
|---|---|
| Has Exit Time | ❌ off |
| Transition Duration | 0 |
| Transition Offset | 0 |

In the **Conditions** list, click **+** and set:

| Parameter | Value |
|---|---|
| `Close` | *(just select the trigger)* |

---

**Transition 4: `Closing` → `Idle_Closed`**

This fires automatically when the closing animation finishes playing.

In the Inspector for this transition:

| Setting | Value |
|---|---|
| Has Exit Time | ✅ on |
| Exit Time | 1.0 |
| Transition Duration | 0 |

Leave the **Conditions** list empty.

---

### Inspector — TrapDoorTrap

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.8–1.5 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |
| **Enemies Can Fall** | ✅ *(enemies walk into the open hole and die)* or ❌ *(enemies path around it)* |
| **Trap Animator** | drag the `Animator` component from `TrapMesh` here |
| **Open Trigger** | `Open` |
| **Close Trigger** | `Close` |

No lever needed for the auto-trigger to fire. The `Close` trigger is used only when a lever closes the hatch externally — see [Type 3](#type-3-lever-controlled-trapdoor) for lever wiring.

!!! tip
    The longer the **Trigger Delay**, the more sadistic the trap — the player hears the hatch creak and knows what's coming but can't step back in time.

---

## Type 3 — Lever-Controlled Trapdoor

**What happens:** The player is standing on a hatch. Someone (or the player themselves) pulls a lever — the hatch opens and they fall. Pulling the lever a second time closes the hatch again.

### Prefab hierarchy

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← hatch mesh + Animator

[somewhere on a wall]
WallLever             ← WallLever (TriggerSource)
```

### Animator

Identical to Type 2 — all four states (`Idle_Closed`, `Opening`, `Idle_Open`, `Closing`), both triggers (`Open`, `Close`), all four transitions with the same settings.

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
3. Click **+** and drag the GameObject that has `TrapDoorTrap` into the slot.

First pull → lever activates → `OnTriggered` → `Open` trigger fires → hatch opens.  
Second pull → lever deactivates → `OnReleased` → `Close` trigger fires → hatch closes.

!!! tip
    Place the lever in the same room as the hatch — the player has to decide whether to stand on the hatch and pull, or investigate first.

---

## Type 4 — Lever Puzzle (Cycling Hatches)

**What happens:** Several hatches block the path. A single lever toggles their state — each pull opens and closes a different combination. The player must find the combination where all hatches on the path are closed and cross safely.

### Setup

Use one `WallLever` (toggle). In the lever's **Targets** list, add all `TrapDoorTrap` components.

Give each hatch a different **Trigger Delay** so they open in sequence, not all at once:

| Hatch | Auto Trigger | Require Party | Trigger Delay | Permanent | Trigger Once |
|---|---|---|---|---|---|
| Hatch A | ❌ | ❌ | 0 s | ❌ | ❌ |
| Hatch B | ❌ | ❌ | 0.3 s | ❌ | ❌ |
| Hatch C | ❌ | ❌ | 0.6 s | ❌ | ❌ |

Lever activates → hatches open one by one. Lever deactivates → they close. The player toggles the lever and watches the rhythm to find the safe crossing window.

!!! note
    **Permanent = false** and **Trigger Once = false** on every hatch — without this they lock open on the first trigger and won't reset.

### Animator

Each hatch: all four states + both triggers + all four transitions — identical to Type 2.

---

## Type 5 — Logic Gate Puzzle (Multi-Lever Lock)

**What happens:** Three levers control one or more hatches. The hatch opens only when all three levers are in the wrong position — the player must find the one safe combination.

### Setup

1. Create an empty GameObject and add a `TriggerLogicGate` component (type: **AND**).
2. Connect all three `WallLever` components as inputs to `TriggerLogicGate`.
3. Connect the `TriggerLogicGate` output to `TrapDoorTrap`.

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

**What happens:** The player stands on a pressure plate — a hatch somewhere else opens. When the player steps off, the hatch closes. The classic "hold the plate to cross" — but the plate opens the hatch under a different player.

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

All four states + both triggers + all four transitions — identical to Type 2.

Player steps on plate → hatch opens. Next time anyone walks over the hatch → they fall.

---

## Type 7 — Repeating Pit

**What happens:** The hatch opens under the player, they fall and die. A few seconds later the hatch closes and resets — ready for the next victim.

Useful in corridors the player must travel through multiple times.

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

### Animator

All four states + both triggers + all four transitions — identical to Type 2.

Sequence: party steps in → after 0.3 s hatch opens → party falls → after 3 s `Close` trigger fires → hatch closes → reset.

---

## Type 8 — Crossbow Corridor

**What happens:** The player enters a corridor, triggers a pressure plate, and an arrow fires from the wall.

No hatch involved — just `ProjectileTrap` + `PressurePlate`.

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

**What happens:** The player steps onto a hatch cell. The hatch opens. At the same moment an arrow fires from the wall. Even if the player avoids the hatch, they still take the arrow.

### Variant A — shared lever

One `WallLever` → **Targets**: both `TrapDoorTrap` and `ProjectileTrap` at the same time.

### Variant B — arrow fires on fall

Use **On Fall Triggered** on `TrapDoorTrap` to call `ProjectileTrap.Trigger()` via UnityEvent. The arrow fires only after the hatch opens.

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

All four states + both triggers + all four transitions — identical to Type 2.

!!! tip
    Add **On Fall Triggered** → `TriggerPlaySound` on each hatch — the sound of the party falling, heard from the far end of the corridor, builds tension for the next attempt.

---

*CrawlerKIT — Mantis3de*
