# Trap Types — Classic Dungeon Traps

Ten guide covers 10 ready-to-use trap configurations built from `TrapDoorTrap` and `ProjectileTrap`. Each type describes what the player sees, what happens, and how to set it up in the Inspector.

---

## Type 1 — Open Pit

**What happens:** The player sees a hole in the floor. They walk in and die instantly.

The simplest possible trap. No `TrapDoorTrap` component needed — the geometry and cell type do all the work.

**Setup:**

1. In the Dungeon Generator, mark the cell as **TrapDoor**.
2. Place the pit mesh (hole, walls on all sides, walls going down).
3. Add no components — done.

The party can enter the cell. Death is instant, no animation, no sound (unless you wire something into **On Fall Triggered**).

---

## Type 2 — Timed Trapdoor

**What happens:** The player steps onto what looks like a normal floor tile. After a moment the hatch opens beneath them and they fall.

The classic hidden trap — the player has no idea it's a trap until it's too late.

**Setup:**

Mesh: closed hatch (looks like a normal floor tile). Animator: states `Idle_Closed` → `Opening` → `Idle_Open`.

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.8–1.5 s *(give the player just enough time to panic before the hatch goes)* |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |
| **Trap Animator** | Animator from `TrapMesh` |
| **Open Trigger** | `Open` |

!!! tip
    The longer the **Trigger Delay**, the crueller the trap — the player has time to realise what's happening but not enough time to back out.

---

## Type 3 — Lever-Controlled Trapdoor

**What happens:** The player is standing on a hatch. Someone (or the player themselves) pulls a lever elsewhere — the hatch opens and they fall.

The classic trust trap — pull the lever and drop.

**Setup:**

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Trigger Delay** | 0.2 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trap Animator** | Animator from `TrapMesh` |

On the lever (`WallLever` or `WallChain`): drag the `TrapDoorTrap` component into its **Targets** list.

!!! tip
    Place the lever in the same room as the hatch — the player has to decide whether to stand on the hatch and pull, or step away first and investigate.

---

## Type 4 — Lever Puzzle (Cycling Hatches)

**What happens:** Several hatches block the path. A single lever toggles their state — each time it's pulled a different combination of hatches is open or closed. The player must find the combination where all hatches on the path are closed and they can cross safely.

**Setup:**

Use one `WallLever` (toggle). Connect several `TrapDoorTrap` components as **Targets**.

Give each hatch a different **Trigger Delay** so they don't all open at once — they open in sequence, creating a shifting "window" of safe tiles:

| Hatch | Auto Trigger | Require Party On Cell | Trigger Delay | Permanent | Trigger Once |
|---|---|---|---|---|---|
| Hatch A | ❌ | ❌ | 0 s | ❌ | ❌ |
| Hatch B | ❌ | ❌ | 0.3 s | ❌ | ❌ |
| Hatch C | ❌ | ❌ | 0.6 s | ❌ | ❌ |

Lever activates → hatches open one by one. Lever deactivates → they close. The player toggles the lever repeatedly and watches for the safe combination.

!!! note
    Set **Permanent = false** and **Trigger Once = false** on every hatch — otherwise they lock open after the first trigger and can't reset.

---

## Type 5 — Logic Gate Puzzle (Multi-Lever Lock)

**What happens:** Three levers control one or more hatches. The hatch opens only when all three levers are in the wrong position — the player must find the one safe combination (all up, all down, or mixed).

**Setup:**

Add a `TriggerLogicGate` component (type **AND**) to a separate GameObject. Connect all three `WallLever` components as inputs. Wire the `TriggerLogicGate` output to `TrapDoorTrap`.

The hatch opens when all levers are active simultaneously. The safe state is any other combination.

You can invert the logic with a **NOT** or **NOR** gate — the hatch starts open and closes only when the player finds the correct combination.

| Field on TrapDoorTrap | Value |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Permanent** | ❌ |
| **Trigger Once** | ❌ |

---

## Type 6 — Pressure Plate Trap

**What happens:** The player stands on a pressure plate — a hatch somewhere else opens. When the player steps off (or something else takes their place on the plate), the hatch closes or opens. The classic "hold the plate to cross" setup — but with a twist: the plate opens the hatch under a second player.

**Setup:**

`PressurePlate` → **Targets** → `TrapDoorTrap`.

| Field on TrapDoorTrap | Value |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Trigger Delay** | 0 s |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |

Player steps on plate → hatch opens. Next time they walk over the hatch → they fall.

---

## Type 7 — Repeating Pit

**What happens:** The hatch opens under the player, they fall and die. A few seconds later the hatch closes and resets — ready for the next victim.

Useful in corridors the player must travel through multiple times (backtracking, respawn points).

**Setup:**

| Field | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ❌ |
| **Close Delay** | 3 s |
| **Trigger Once** | ❌ |
| **Trap Animator** | Animator from `TrapMesh` |

---

## Type 8 — Crossbow Corridor

**What happens:** The player enters a corridor, triggers a pressure plate, and an arrow fires from the side.

The classic dart/arrow corridor — no hatch involved, just `ProjectileTrap` + `PressurePlate`.

**Setup:**

`PressurePlate` → **Targets** → `ProjectileTrap`.

| Field on ProjectileTrap | Value |
|---|---|
| **Projectile Prefab** | arrow / bolt prefab |
| **Fire Direction** | perpendicular to the player's movement |
| **Damage** | 20–30 |
| **Aim Target** | Transform from `PartyVisuals` |

Place multiple `ProjectileTrap` components on both sides of the corridor with opposing **Fire Direction** values — arrows fire from left and right simultaneously.

---

## Type 9 — Pit + Crossbow Combo

**What happens:** The player steps onto a hatch cell. The hatch opens beneath them. At the same moment an arrow fires from the side. Even if the hatch is lever-controlled and the player avoids standing on it, they still take the arrow.

`TrapDoorTrap` and `ProjectileTrap` triggered from the same source.

**Setup — shared lever:**

One `WallLever` → **Targets**: both `TrapDoorTrap` and `ProjectileTrap` at the same time.

**Setup — arrow fires on fall:**

Leave the lever off. Use **On Fall Triggered** on `TrapDoorTrap` to call `ProjectileTrap.Trigger()` via UnityEvent. The arrow fires only after the hatch opens.

| Field on TrapDoorTrap | Value |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **On Fall Triggered** | → `ProjectileTrap.Trigger()` |

| Field on ProjectileTrap | Value |
|---|---|
| **Fire Direction** | from the side |
| **Damage** | 25 |

---

## Type 10 — Gauntlet

**What happens:** The player must cross a corridor packed with hatches and arrows. Hatches open in sequence (each with a different Trigger Delay), arrows fire from a pressure plate at the corridor entrance. A safe path exists — the player has to find it.

Combines Type 4 (cycling hatches), Type 7 (repeating), and Type 8 (crossbow corridor).

**Setup:**

1. Lay out 4–6 `TrapDoor` cells along the corridor — each with a different **Trigger Delay** (0 s, 0.5 s, 1 s, 1.5 s…), **Permanent = false**, **Trigger Once = false**.
2. Connect one `WallLever` to all hatches — toggling it shifts which hatch is open at any given moment.
3. Place a `PressurePlate` at the corridor entrance wired to `ProjectileTrap` components on both sides.
4. The player reads the hatch rhythm first, then enters and sprints through the safety window.

!!! tip
    Add **On Fall Triggered** → `TriggerPlaySound` on each hatch — the sound of the party falling heard from the far end of the corridor builds tension for the next attempt.

---

*CrawlerKIT — Mantis3de*
