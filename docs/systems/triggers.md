# Trigger System

The Trigger System is the backbone of interactive dungeon elements — doors, levers, pressure plates, traps and everything else the player or the environment can activate. It lives in the `CrawlerKitTriggers` module and follows a simple **source → target** pattern: a source fires when the player interacts with it (or steps on it), and each target reacts by opening a door, spawning a projectile, playing a sound, and so on.

Sources and targets are connected by direct `TriggerTarget` references set in the Inspector — there are no string IDs or lookup tables involved in routing. String IDs (`sourceId`, `blockerId`, `targetId`) exist only so the Save System can persist state across sessions.

---

## Core concepts

### TriggerSource

`TriggerSource` is the base class for every interactive object. It holds the list of connected targets and exposes the `Interact()` entry point that `InteractionSystem` calls when the player clicks on it.

**Inspector fields (all sources inherit these):**

| Field | Description |
|---|---|
| **Source Id** | Unique string identifier used by the Save System. Must be unique per scene. Leave empty if this source doesn't need to be saved (e.g. a one-shot button whose state doesn't matter on reload). |
| **Interact From Adjacent Cell** | When **disabled** (default), the player must be standing on the exact same grid cell as the trigger to interact with it. When **enabled**, the player can also interact from the cell directly in front of them — useful for buttons mounted on the far side of a wall that the player faces from their own cell. See [Interaction range](#interaction-range) for the full rules. |
| **Targets** | Direct references to one or more `TriggerTarget` components this source will notify when activated or deactivated. |
| **Interact Sound** | Audio clip played when the player successfully interacts with this trigger (button press, lever pull, key insert, etc.). Leave empty for no sound. |
| **Interact Volume** | Volume of the interact sound (0–1). |
| **Interact Sound 2D** | When enabled, the sound plays as a flat 2D clip with no spatial falloff — equally audible regardless of distance or position. Useful for UI-like triggers (wall buttons, chains) where 3D positioning would make the sound feel too quiet when standing close to a wall. When disabled (default), the sound plays at the trigger's world position as a 3D positional clip. |

!!! note "Sound-before-activate ordering"
    All built-in sources (WallButton, WallLever, WallChain, WallKeyhole, WallAlcove, WallMultiSwitch, PressurePlate) play the interact sound first, then wait for the clip to finish before notifying targets. This prevents the door's open sound from immediately drowning out the trigger sound. If **Interact Sound** is empty the activation fires on the same frame as usual.

### TriggerTarget

`TriggerTarget` is the base class for every reactive object. It receives `OnTriggered` / `OnReleased` calls from its source and acts accordingly. Targets never know which source fired them — they simply respond to activate and deactivate events.

### Interaction range

The `InteractionSystem` (the component that handles mouse clicks and routes them to triggers) applies the following range rules for every `TriggerSource`:

1. **Same cell** — the player is always in range, regardless of the **Interact From Adjacent Cell** setting.
2. **Adjacent cell (front)** — the player can reach the trigger from the cell directly ahead of them only if **Interact From Adjacent Cell** is enabled on that source and there is no closed, non-passable door or blocker between the two cells.
3. **Any other position** — not in range; the click is ignored.

This means two triggers placed on the same tile can have different reach rules independently of each other — toggle a lever on one and enable adjacent reach on a button next to it, or vice versa.

!!! tip "When to enable Interact From Adjacent Cell"
    Enable it on buttons that are physically mounted on a wall between two cells, where the player naturally stands in one cell and reaches the button across the edge. Leave it disabled for levers, chains and other triggers that are physically within the player's cell (on the floor or on a nearby column), so the player must walk up to them before interacting.

---

## Sources

### Wall Button — `WallButton`

A one-shot button that fires on press and stays down until the connected door closes. Once pressed it ignores further clicks until `NotifyDoorClosed()` is called on it (which `TriggerOpenDoor` and `TriggerOpenDoorTimed` do automatically when their door finishes closing).

Internally it pulses — it activates its targets, waits a brief moment, then deactivates them — so targets receive both the `Triggered` and `Released` events in quick succession. Targets that only care about activation (most doors) simply ignore the release.

| Inspector field | Description |
|---|---|
| **Pulse Time** | Seconds between `Activate` and `Deactivate` inside the pulse. Default 0.3 s. Increase slightly if the door's open animation takes a frame or two to read the trigger. |
| **Button Animator** | Animator on the button mesh. |
| **Press Anim Trigger / Release Anim Trigger** | Animator trigger names for the press-down and spring-back animations. |

### Wall Lever — `WallLever`

A toggle lever: first interact activates, second interact deactivates, and so on. Unlike a button it doesn't spring back — it stays in whatever position it was last set to, which is also what the Save System restores.

If a connected target refuses the incoming state change (for example, a door refuses to close because an enemy is standing in it) the lever stays in its current position and plays no animation. The player must wait until the doorway is clear and try again.

| Inspector field | Description |
|---|---|
| **Lever Animator** | Animator on the lever mesh. |
| **Activate / Deactivate Trigger** | Animator trigger names. |
| **Activated / Deactivated State Name** | Animator state names used by the Save System to snap the lever to the correct visual on load. |

### Wall Chain — `WallChain`

Functionally identical to `WallLever` (a toggle) but styled as a pull-chain. Use whichever fits the visual theme of the dungeon.

### Wall Alcove — `WallAlcove`

A wall recess that holds an item. The player interacts with it by having an item on the cursor (to place) or by clicking when the alcove already holds an item (to retrieve). When an item is placed into an alcove that is connected to targets, those targets are activated; when the item is removed they are deactivated. Use this for item-based puzzles — place the correct amulet in the alcove to open a door.

### Wall Keyhole — `WallKeyhole`

A keyhole that accepts a specific item from the cursor as a key. When the correct key is inserted the targets activate. The key is consumed in the process. Unlike other sources, the keyhole always requires the player to be on the **same cell** — the **Interact From Adjacent Cell** setting has no effect on keyholes.

Configure which item acts as the key via **DoorKeyholeSetup**, a companion component that links the keyhole to a specific `ItemData` asset.

### Wall Multi-Switch — `WallMultiSwitch`

A switch with multiple positions (rather than a simple on/off toggle). Each position can activate a different set of targets, making it useful for puzzles where the player must select one of several options — for example, rotating a dial to choose a spell element.

### Pressure Plate — `PressurePlate`

A floor tile that activates when the player (or a world item) is on it and deactivates when the tile becomes empty. It polls its own cell automatically on every frame something could have changed — no explicit player interaction is needed.

Pressure plates work with items: drop an item on the plate and it stays activated. This enables weight-based puzzles. A plate connected to a `TriggerOpenDoor` with **Close On Release** enabled will keep the door open for as long as something is on it, and let it close the moment the tile clears.

---

## Targets — Doors

### TriggerOpenDoor

The standard interactive door. Opening and closing are driven by animator triggers, and the physical blocker state (which the grid uses to determine movement) is updated through `IGridSystem.TryOpenDoor` / `TryCloseDoor`.

| Inspector field | Description |
|---|---|
| **Blocker Id** | The `blockerId` from the `EdgeBlockerMarker` on this door. Must match exactly — the grid uses this to find the right blocker. |
| **Close On Release** | When enabled, the door closes as soon as the source releases (lever pulled back, pressure plate emptied). When disabled, the door stays open until something else closes it. |
| **Skip Close Guard** | Skip the occupancy check when closing. Enable this for toggle doors where the lever is on the same cell as the door, because the guard would wait forever (the player is always on that cell). |
| **Button Release Delay** | Seconds to wait after the close animation starts before telling the connected `WallButton` to spring back. Set this to roughly the length of your Close animation so the button returns to idle only after the door is fully shut. |
| **Open Sound** | Audio clip played at the door's world position when it opens. 3D spatial — audible from a distance. |
| **Close Sound** | Audio clip played when the door closes. |
| **Door Sound Volume** | Volume for both open and close sounds (0–1). |
| **Door Animator** | Animator on the door mesh. |
| **Open / Close Anim Trigger** | Trigger names for the open and close animations. |
| **Opened / Closed State Name** | Animator state names used by the Save System to snap the door to its saved visual on load. |

**Close guard behaviour.** When **Close On Release** is enabled and the player releases the source, the door does not close immediately. Instead it waits until both grid cells the door connects are clear of the player and of any living enemy, then closes. This prevents the door from visually clipping through a party member or an enemy that walked into the doorway while it was open. The check runs every 0.1 seconds.

!!! note "Enemy blocking"
    An enemy standing in the doorway prevents the door from closing, exactly as the player does. This is intentional — the enemy walked in because the player opened the door, so the door waits for them to leave. Once the enemy clears the cell the door closes normally.

    In addition, if the source is a lever or button with **Close On Release** enabled, clicking it while an enemy is standing in the doorway is refused entirely — the lever does not move and no animation plays. The doorway must be clear before the player can initiate a close.

### TriggerOpenDoorTimed

A door that opens on trigger and closes automatically after a fixed delay. Useful for portcullises, countdown traps and any door that should not stay open indefinitely.

| Inspector field | Description |
|---|---|
| **Blocker Id** | Same as `TriggerOpenDoor`. |
| **Close Delay** | Seconds the door stays open before beginning to close. Default 3 s. |
| **Door Animator** | Animator on the door mesh. |
| **Open / Close Anim Trigger** | Trigger names. |

After `closeDelay` seconds the door applies the same close guard as `TriggerOpenDoor` — it waits until the player and any enemy have left the doorway before actually closing.

### TriggerDirectionalDoor

A door that only opens when the player approaches from one specific direction. Useful for one-way passages, secret exits and anti-backtrack mechanisms.

---

## Targets — Traps

### ProjectileTrap

Fires a projectile (an arrow, a dart, a magical bolt) at the party when activated. The projectile travels in a straight line and deals damage to the party on hit.

| Inspector field | Description |
|---|---|
| **Trap Id** | Unique identifier for Save System persistence. |
| **Projectile Prefab** | The projectile to spawn. Must have a `TrapProjectile` component. |
| **Fire Direction** | Which direction the projectile travels: North, South, East or West. |
| **Damage** | Damage dealt to the party on hit. |

### TrapDoorTrap

`TrapDoorTrap` covers two distinct scenarios that share the same component: an **open pit** the party can see and walk into, and a **hidden trapdoor** that opens beneath their feet. The difference is entirely in the Dungeon Generator cell type and whether an animator is attached.

**Open pit (visible, instantly deadly)** — the most common case. The party can see a hole in the floor; walking into it kills everyone.

1. In the Dungeon Generator, mark the cell as **TrapDoor** (orange). Do not use **Pit** — that type is impassable like a wall and the party can never enter it.
2. Place your pit mesh at that cell (a floor tile with a hole, a dark void, etc.).
3. Add a `TrapMarker` and a `TrapDoorTrap` component to the prefab root.
4. In `TrapDoorTrap`: set **Damage Preset** to `InstantKill`, leave **Trigger Delay** at 0, enable **Auto Trigger On Enter**, enable **Permanent**, leave **Trap Animator** empty.
5. No `TriggerSource` is needed — the trap fires the moment the party steps onto the cell.

| Inspector field | Description |
|---|---|
| **Auto Trigger On Enter** | Must be enabled. The trap fires automatically when the party enters the cell. |
| **Trigger Delay** | Seconds between entering the cell and applying damage. 0 = instant death on step. |
| **Damage Preset** | `InstantKill` kills every party member immediately (game over). `Heavy` deals 50, `Light` deals 20, `Custom` uses the value in **Fall Damage**. |
| **Permanent** | Keep enabled for an open pit — the hole stays open. Disable only for a trapdoor that closes again. |
| **Trigger Once** | Keep enabled. The pit does not need to fire multiple times. |
| **Trap Animator** | Leave empty for an open pit. Only needed for an animated trapdoor that visually opens. |
| **On Fall Triggered** | Unity event fired just before damage is applied. Wire a sound or camera shake here. |

**Hidden trapdoor (animated, opens under the party)** — the classic dungeon surprise. The floor looks normal; when the party steps on it a hatch opens and they fall.

Setup is identical to the open pit but with an `Animator` assigned to **Trap Animator** that has an `Open` trigger (and optionally a `Close` trigger). Set **Trigger Delay** to 0.2–0.4 s so the hatch animation has time to play before damage lands. Set **Permanent** to false if the hatch should close again afterward.

!!! note "Pit vs TrapDoor cell type"
    **Pit** (`GridFlagExtended → Cell Type → Pit`) marks an impassable cell — the party and enemies are blocked from entering it, exactly like a wall. Use it for decorative chasms the player looks into but cannot cross.

    **TrapDoor** is a normal walkable floor cell that the party *can* enter. The `TrapDoorTrap` component reacts when they do. Use TrapDoor for any hole the party should be able to fall into.

### TrapProjectile

The companion component to `ProjectileTrap`. Attach it to the projectile prefab. It handles movement, collision detection and applying damage when the projectile reaches the party's cell.

---

## Targets — Walls

### TriggerIllusoryWall

A wall that appears solid but can be walked through when revealed. Connected to a `WallCore` component. When activated the wall is flagged as passable; when deactivated it becomes solid again.

The wall system supports several reveal behaviours that control *when* the wall reveals itself, all implemented as components alongside `WallCore`:

| Behaviour | Trigger |
|---|---|
| **ProximityRevealBehaviour** | Reveals when the party steps within a set number of cells. |
| **DelayedProximityBehaviour** | Same as proximity, but with a configurable delay before reveal. |
| **TimedRevealBehaviour** | Reveals after a fixed time from scene load. |
| **AmbushRevealBehaviour** | Reveals when a connected enemy spawner activates (the enemy "bursts through" the wall). |
| **CounterRevealBehaviour** | Reveals after the party has passed the tile a set number of times. |
| **ProjectileRevealBehaviour** | Reveals when hit by a projectile. |
| **PhaseWallBehaviour** | Allows the party to walk through without revealing the wall visually — they simply pass through. |
| **OneWayPassageBehaviour** | Passable from one direction only. |
| **BiomeWallModifier** | Changes the wall's visual appearance to match a different biome theme without affecting behaviour. |

### WallCore

The central component that manages a wall's reveal state and routes it through the behaviour pipeline. Every wall behaviour component on the same GameObject communicates through `WallCore`. You rarely need to interact with it directly — configure the individual behaviours instead.

---

## Targets — Events

### TriggerChain

Forwards activation and deactivation to a list of other `TriggerTarget` components. Use it to fan out one source to many targets without wiring every target directly to the source.

### TriggerLogicGate

A logic gate (AND, OR, NOT, XOR) with multiple `TriggerSource` inputs. The gate activates its targets only when the logical condition across all inputs is satisfied. Useful for puzzles where multiple levers must all be in the correct position before a door opens.

### TriggerSequenceGate

A sequence gate that requires its connected sources to be activated in a specific order. Wrong order resets the sequence. Use it for combination lock puzzles.

### TriggerRelay

Listens to `OnActivated` / `OnDeactivated` events from a source and re-emits them to targets. Similar to `TriggerChain` but driven by C# events rather than direct calls, which can be useful when sources and targets are in different parts of the scene hierarchy.

### TriggerShowMessage

Displays a text message on screen when activated. Use for signs, lore pickups or tutorial hints wired to a pressure plate or trigger zone.

### TriggerPlaySound

Plays an audio clip when activated. Optionally plays a different clip on deactivation.

---

## Wiring a door — step by step

1. Place your door prefab in the scene. It should have an `EdgeBlockerMarker` component with a filled-in **Blocker Id** field. If the field is empty, run **Generate Unique IDs (all doors)** from the Dungeon Generator.
2. Add a `TriggerOpenDoor` component to the door GameObject. Set **Blocker Id** to the same value as the `EdgeBlockerMarker`.
3. Configure the animator fields to point to your door's `Animator` and the correct trigger/state names.
4. Place your lever (or button, or pressure plate) in the scene. Add the appropriate source component.
5. Drag the door's `TriggerOpenDoor` component into the source's **Targets** list.
6. Fill in the source's **Source Id** field with a unique string.
7. Press Play and test.

If the door should close automatically after a delay, use `TriggerOpenDoorTimed` instead and skip the **Close On Release** option — the timed door handles its own close timer.

---

## Save System integration

Every `TriggerSource` that has a non-empty **Source Id** registers itself with the Save System. On save, each source writes its `IsActivated` boolean. On load, each source restores that boolean and calls `RestoreState` on all its targets, which snap their visual state (animator, blocker open/closed) to match without firing any events.

Door targets also read the actual `EdgeBlocker.isOpen` state from the grid JSON on restore, so the physical passability is always authoritative — the visual snaps to match it, rather than the other way around.

---

*CrawlerKIT — Mantis3de*
