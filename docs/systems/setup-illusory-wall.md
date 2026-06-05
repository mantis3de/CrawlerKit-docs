# Setup Guide — Illusory Wall

An illusory wall looks exactly like a solid wall but the party can walk through it. What makes it passable is a single flag on its `EdgeBlockerMarker` — **Is Passable**. Everything else (the dissolve animation, the trigger logic, whether it reveals at all) is optional layered on top.

---

## How the grid sees it

Every wall in CrawlerKit is registered as an `EdgeBlocker` in the grid. Whether a wall is physically passable is determined entirely by **Is Passable** on `EdgeBlockerMarker`. Visibility — whether the wall mesh is showing — is controlled separately by `WallCore` and the behaviours on top of it. A wall can look solid but be passable, or look invisible but block movement.

!!! note "Dungeon Generator"
    Secret walls are placed from the **Secret Wall Library** panel in the Dungeon Generator. After placing, run **Generate Unique IDs (all walls)** so each wall gets a unique ID before JSON export.

---

## Core components

### EdgeBlockerMarker

Registers the wall with the grid. Required on every wall.

| Field | Value for illusory walls |
|---|---|
| **Blocker Id** | Unique ID, e.g. `wall_secret_01` |
| **Is Passable** | ✅ — must be enabled for the party to walk through |
| **Is Open** | Leave disabled |

### WallCore

The central controller for walls that change their appearance. Manages the reveal/restore pipeline and drives the mesh and collider. Use this for walls driven by **behaviour components** (proximity, timed, ambush, etc.).

| Field | Description |
|---|---|
| **Wall Mesh** | The wall's mesh child. |
| **Wall Collider** | Physical collider on the root. Leave empty if there is no collider. |
| **Collides When Hidden** | Keep disabled — illusory walls should not block movement even when visible. |
| **Reveal Duration** | Seconds to match the dissolve animation length. Set to 0 for an instant pop. |
| **Restore Duration** | Seconds for the restore animation. |
| **Starts Revealed** | Enable if the wall should start invisible (useful for re-appearing walls). |

### TriggerIllusoryWall

The `TriggerTarget` variant of wall control. Use this when a `TriggerSource` (lever, button, plate, logic gate) drives the reveal. Receives `OnTriggered` / `OnReleased` and plays the reveal/restore animations accordingly.

| Field | Description |
|---|---|
| **Blocker Id** | Must match `EdgeBlockerMarker.blockerId`. |
| **Wall Mesh** | The wall's mesh child. |
| **Wall Animator** | Animator from the mesh child. |
| **Reveal / Restore Anim Trigger** | Animator trigger names. |
| **Dissolve Delay** | Seconds after the reveal trigger before the mesh is hidden. Match to animation length. |

---

## Behaviour components (used with WallCore)

Add these alongside `WallCore` to define *when* the wall reveals itself. Multiple behaviours can coexist on the same wall.

| Behaviour | Reveals when… |
|---|---|
| `ProximityRevealBehaviour` | Party steps onto the wall's cell. |
| `DelayedProximityBehaviour` | Party steps onto the cell, but after a configurable delay. |
| `TimedRevealBehaviour` | A fixed time after scene load. |
| `AmbushRevealBehaviour` | A connected enemy spawner activates — the enemy bursts through. |
| `CounterRevealBehaviour` | The party has passed through or entered the cell a set number of times. |
| `ProjectileRevealBehaviour` | A projectile hits the wall. |
| `PhaseWallBehaviour` | Party walks through without any visual change — the wall stays solid-looking. |
| `OneWayPassageBehaviour` | Passable from one direction only. |
| `BiomeWallModifier` | Changes the wall's visual to match a different biome. Not a reveal — purely cosmetic. |

---

## Wall Animator setup

If you want a dissolve animation, the mesh child's Animator Controller needs these states and triggers:

```
States
  Idle_Hidden     ← default (wall looks solid)
  Revealing       ← dissolve playing
  Idle_Revealed   ← wall invisible / gone
  Restoring       ← restore playing (only if wall can restore)

Triggers
  Reveal
  Restore
```

Transitions: `Idle_Hidden → Revealing` on trigger `Reveal` (no Exit Time). `Revealing → Idle_Revealed` on Exit Time. Same pattern for Restore.

!!! tip "No animator needed"
    Skip the Animator entirely if you want the wall to simply disappear. Set **Reveal Duration** to `0` on `WallCore` and the mesh hides immediately on reveal.

---

## Map behaviour

Secret floor cells (marked as Secret in the Dungeon Generator) never appear on the dungeon map until explored. Illusory walls themselves don't affect the map — only the floor cell type on the other side does. A room behind a secret wall with normal floor tiles appears on the map as soon as the party enters. Mark those floor tiles as Secret to keep the room hidden until explored.

---

## Component summary

| Goal | Components on root |
|---|---|
| Walk through, no visual | `EdgeBlockerMarker` (Is Passable ✅) |
| Reveals on approach | `EdgeBlockerMarker`, `WallCore`, `ProximityRevealBehaviour` |
| Reveals on trigger | `EdgeBlockerMarker`, `TriggerIllusoryWall` |
| Reveals on timer | `EdgeBlockerMarker`, `WallCore`, `TimedRevealBehaviour` |
| Enemy bursts through | `EdgeBlockerMarker`, `WallCore`, `AmbushRevealBehaviour` |
| One-way passage | `EdgeBlockerMarker`, `WallCore`, `OneWayPassageBehaviour` |

---

*CrawlerKIT — Mantis3de*
