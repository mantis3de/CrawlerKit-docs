# CrawlerKIT — Enemy Spawner

## Overview

The Enemy Spawner is the tool you use to drop enemies into your dungeon and assign their loot — without touching the Inspector. You open it from Unity's top menu under **CrawlerKit → Enemy Spawner**.

The window has three columns: a **Presets** list on the left, a **Loot Tables** browser in the centre, and the **Enemies** working area on the right. The short version: you build a working list of enemy prefabs, drag loot tables onto them, place them into the scene with one click, and save your favourite groups as reusable presets.

The Enemy Spawner pairs with two runtime pieces: each enemy prefab carries an **EnemySpawner** component (which holds an **EnemyData** asset describing the creature) and the loot tables you assign come from the Inventory Editor. The window reads the enemy's stats straight from its EnemyData so you can see exactly what you are placing.

---

## Presets (left column)

A preset is a saved group of enemies you can recall instantly — handy for recurring encounters like "Goblin patrol" or "Crypt pack". The panel lists every preset you've created, each showing its name and an enemy count, and clicking a preset loads its enemies into the right-hand Enemies list.

To create one, build your enemy list on the right and click the green **+** at the top of the Presets panel (or the dedicated save action). The editor suggests a name based on the first one or two enemy types in the list, which you can edit before confirming. The red **✕** on a preset row deletes it after a confirmation prompt. If you have no presets yet, the panel shows a short hint inviting you to create your first.

Presets and the current working list are stored automatically between Unity sessions, so your setup is preserved when you close and reopen the window.

---

## Loot Tables Browser (centre column)

The centre column lists every **LootTableData** asset in your project, sorted by name. Each row shows the table name, a roll-mode badge (**Pick 1**, **Pick N–N**, or **Roll All**), and up to five mini item icons previewing its contents, with a "+N" indicator when there are more. Hovering a row reveals a tooltip with the entry count and the table's overall drop chance.

The whole point of this column is drag-and-drop: pick up any loot table and drop it onto an enemy in the right column to assign it. A small hint above the list ("Drag onto enemy →") reminds you of this. A **↺** refresh button at the top re-scans the project for loot tables — useful after you create new ones in the Inventory Editor without closing the window. If no loot tables exist, the panel points you to the Inventory Editor to create them.

---

## Enemies (right column)

This is the working area where you assemble the enemies you want to place. Click the green **+** at the top to add an empty enemy row, and use the **✕** on any row to remove it. Rows can be reordered by dragging the **≡** handle on the left — a blue line shows where the row will drop.

Each enemy row contains:

- **Prefab preview** — a 96×96 thumbnail of the assigned prefab (or "No Preview" when empty).
- **Enemy name** — read from the EnemyData's name, falling back to the prefab name.
- **Object field** — assign the enemy prefab here, or change it later.
- **Stats panel** — a compact read-out of the enemy's EnemyData (see below).
- **Loot table drop zone** — drop a loot table from the centre column here.
- **+ ADD** button — places the enemy into the scene.
- **✕** button — removes the row.

### Enemy Stats Panel

When a prefab with EnemyData is assigned, the row shows the creature's key stats as small coloured pills so you can compare enemies at a glance. The first row covers core combat values:

- **HP** — maximum health.
- **ATK** — attack damage.
- **DEF** — defense.
- **EVA** — evasion, shown as a percentage (only when above zero).
- **VIS** — detection range, how far the enemy can notice the party.
- **RNG** — attack range (only shown for ranged enemies, when greater than 1).

The second row shows the enemy's **behaviour** (colour-coded) plus its movement and attack cooldowns (for example "mv 1.0s atk 1.5s"), and a **STATIONARY** marker if the enemy cannot move. Behaviour types include Aggressive, Stationary, Patrol, PatrolRoute, Coward, Ranged and Passive. If no EnemyData is assigned the panel simply prompts you to assign a prefab.

### Assigning a Loot Table

Drag a table from the centre browser onto the green drop zone in the row. The zone turns green and shows the assigned table's name with mini item icons. The assignment is written directly to the enemy's EnemyData asset (its `lootTable` field) and saved, so every instance of that enemy will use it. A small **✕** inside the zone clears the assignment, and hovering shows a tooltip with the table's entry count and drop chance.

### Placing an Enemy in the Scene

Click the green **+ ADD** button to drop the enemy into the current scene. The prefab is instantiated at the origin (position zero), selected, and framed in the Scene view so you can immediately move it to the right tile. The placement is registered with Undo, so Ctrl+Z removes it cleanly. The button is disabled until a prefab is assigned.

---

## How the Enemy Behaves at Runtime

The EnemySpawner component placed in the scene controls how and when the enemy appears in play. Its position determines the grid tile the enemy spawns on, and you can set the following on the component:

- **Enemy** — the EnemyData asset and a **Start Facing** direction (North, South, East, West).
- **Spawner ID** — a unique identifier for save/load, auto-generated if left empty.
- **Respawn Delay** — seconds before the enemy respawns after death; 0 means it never respawns.
- **Initial Delay** — seconds to wait before the first spawn.
- **Patrol Route** — a list of waypoint Transforms the enemy walks between (used by the PatrolRoute behaviour), with a **Patrol Mode** of either **Loop** (A→B→C→A) or **PingPong** (A→B→C→B→A).

The spawner integrates with the save system: an enemy that was killed and saved as dead will not respawn on load, and a saved enemy is placed directly at its saved cell, facing and health, avoiding any visible pop or rotation snap. In the Scene view the spawner draws a gizmo — a red sphere for the spawn point, a yellow facing arrow, the enemy name (with respawn time if set), and a cyan line connecting any patrol waypoints.

---

## Behavior types

Each enemy's behavior is set in its EnemyData asset. The behavior pill in the Enemies column is color-coded for quick identification.

**Aggressive** — chases the party as soon as it detects them (within Detection Range tiles) and does not give up until one side is dead. The most common behavior for dungeon monsters.

**Stationary** — stands on its spawn tile and never moves. Attacks when the party enters its attack range. Useful for turret-style guardians and immobile bosses.

**Patrol** — walks a random route near its spawn point (within Patrol Radius tiles) when the party is not detected. Switches to Aggressive chase on detection.

**PatrolRoute** — follows the waypoints assigned in the EnemySpawner's Patrol Route list, looping or ping-ponging between them. Switches to Aggressive chase on detection. Requires at least one waypoint Transform set in the spawner.

**Coward** — fights normally until its HP drops below the Flee Threshold (default 30%). Once below that threshold it runs away from the party. Useful for archers and fragile casters that flee when cornered.

**Ranged** — keeps its distance and attacks from range. Flees if the party steps into the adjacent cell. Attack Range on the EnemyData must be greater than 1 for the ranged attack to reach.

**Passive** — wanders randomly and ignores the party unless attacked first. Once hit, switches to Aggressive for the rest of the encounter. Useful for neutral creatures or animals.

---

## EnemyData asset reference

EnemyData is a ScriptableObject that defines a creature type. Create one via `Assets → Create → CrawlerKit → Enemy Data`. Every EnemySpawner in the scene references one.

### Identity

| Field | Description |
|---|---|
| **Enemy Name** | Display name shown in the spawner window and in UI. |
| **Description** | Free-text lore or notes. Not shown at runtime. |

### Stats

| Field | Description |
|---|---|
| **Max HP** | Maximum health points. |
| **Attack Damage** | Damage dealt per attack, modified by party defense via CombatFormulas. |
| **Defense** | Reduces incoming damage. |
| **Evasion** | Chance (0–0.6) to dodge an incoming attack. Interacts with attacker DEX. |

### Behavior

| Field | Description |
|---|---|
| **Behavior** | AI pattern — see Behavior types above. |
| **Move Cooldown** | Seconds between movement steps. Lower = faster enemy. |
| **Attack Cooldown** | Seconds between attacks. |
| **Attack Connect Time** | Delay in seconds from the start of the attack animation to when damage lands. Match this to the visual hit moment in the animation clip. |
| **Camera Shake Delay** | Seconds from the damage landing moment to the camera shake. Set to 0 for an immediate shake on hit. |
| **Detection Range** | Tiles away at which the enemy can detect the party. |
| **Attack Range** | 1 = melee (adjacent cell only). 2 or higher = ranged attack reaching that many tiles. |
| **Can Move** | Uncheck to make the enemy stationary regardless of behavior. |
| **Patrol Radius** | For Patrol behavior — how far from spawn the enemy roams. |
| **Flee Threshold** | For Coward behavior — HP fraction (0–1) below which the enemy flees. |

### Death

| Field | Description |
|---|---|
| **Death Linger Duration** | Extra seconds before the corpse is destroyed after the death animation ends. 0 = destroyed immediately. |

### Visual

| Field | Description |
|---|---|
| **Speed Multiplier** | Scales the visual move speed. 1 = synced to the Walk clip. 2 = twice as fast visually. |
| **Move Start Delay** | Seconds after the Walk animation starts before the enemy physically steps to the new cell. Tune if the enemy appears to walk in place before moving. |
| **VFX Impact Forward Offset** | Pulls the impact point toward the caster so the effect lands on the model surface rather than inside it. |
| **Health Bar Scale** | Size multiplier for the HP bar. |
| **Health Bar Max Distance** | The HP bar becomes invisible when the camera is farther than this many units away. |

### Health Bar and VFX aim — prefab child GameObjects

The HP bar position and the spell VFX aim point are no longer set as numeric offsets on EnemyData. Instead, you place empty child GameObjects directly in the enemy prefab and position them visually in the Scene view.

**HealthBarAnchor** — add an empty child to the enemy prefab and attach the `HealthBarAnchor` component. The HP bar will appear at exactly this transform's world position. In the Scene view, the component draws a red bar-shaped gizmo so you can see where the bar will float. Drag the object up to the top of the model's head. No `HealthBarAnchor` = no HP bar shown at runtime.

**VFXAimPoint** — add an empty child and attach the `VFXAimPoint` component. Spell projectiles and impact VFX aim at this transform's world position. The gizmo shows a cyan sphere and crosshair. Place it at chest height on the model. If no `VFXAimPoint` is present, the spell falls back to 1 unit above the model pivot.

### Animation

These fields must match the parameter and state names in the enemy's Animator Controller exactly.

| Field | Default | Description |
|---|---|---|
| **Is Walking Bool** | `IsWalking` | Bool parameter — true while moving. |
| **Attack Trigger** | `Attack` | Trigger for the attack animation. |
| **Hit Trigger** | `Hit` | Trigger for the damage/flinch animation. |
| **Death Trigger** | `Death` | Trigger for the death animation. |
| **Walk Clip Name** | `Walk` | Animator state name for the walk animation. |
| **Attack Clip Name** | `Attack` | Animator state name for the attack animation. |
| **Death Clip Name** | `Death` | Animator state name for the death animation. |

### Audio

| Field | Description |
|---|---|
| **Attack Sound** | Played when the enemy attacks. |
| **Attack Sound Delay** | Seconds between the attack animation starting and the attack sound playing. The clip starts on frame 0 but the paw/claw connects partway through — set this to the moment of impact so the sound lands on the hit, not before the swing. `0` = play immediately. Default 0.3. This is the audio counterpart of **Attack Connect Time** (which delays the *damage*); tune both to the same visual hit moment. |
| **Hurt Sound** | Played when the enemy takes damage. |
| **Death Sound** | Played when the enemy dies. |
| **Spatial Audio** | When enabled, sounds fade with distance (3D spatial audio). When disabled, sounds always play at full volume regardless of how far the enemy is. |
| **Audio Min Distance** | Distance in world units at which the sound plays at full volume. Only applies when Spatial Audio is enabled. Default 15 (3 cells at cell size 5). |
| **Audio Max Distance** | Distance in world units at which the sound becomes inaudible. Only applies when Spatial Audio is enabled. Default 40 (8 cells). |

### Loot

| Field | Description |
|---|---|
| **Loot Table** | The LootTableData asset dropped on death. Assign here or via drag-and-drop in the Enemy Spawner window. |

---

## Party audio — footsteps and hit sounds

The `PartyVisuals` component (on your camera or party root GameObject) handles all movement and feedback animations. It now also drives two categories of party-side audio.

**Footsteps** — assign one or more AudioClips to the **Footstep Sounds** array. Each time the party moves one cell a random clip from the array is chosen and played. Using several variants prevents the sound becoming repetitive. **Footstep Volume** controls the level (0–1).

**Party Hit Sound** — the clip assigned to **Party Hit Sound** plays whenever an enemy's attack successfully lands on any party member. Use it to communicate that the party took damage without relying solely on the HP bar. **Party Hit Volume** controls the level.

Both sounds play through a 2D persistent AudioSource on the PartyVisuals GameObject (no distance falloff — they are player-feedback sounds, not world events).

---

## Weapon and hand-slot audio

The `HandSlotUI` component has two extra audio fields in the **Audio** section.

**Default Swing Sound** plays when the player attacks with a weapon that has no `Attack Sound` set in its `ItemData`. It acts as a fallback so every melee attack makes some sound even before per-weapon audio is configured.

**Miss Sound** plays when the attack misses entirely (the animation still fires but no damage lands). **Hand Sound Volume** controls the level of both.

Per-weapon sounds are set on the `ItemData` asset — see the Inventory Editor docs for the full Audio section reference.

---

## Workflow Summary

1. Open **CrawlerKit → Enemy Spawner**.
2. In the Enemies column, click **+** and assign an enemy prefab; repeat for each enemy you want available.
3. Review each enemy's stats in the row's stats panel.
4. Drag a loot table from the centre browser onto an enemy's drop zone to assign its drops.
5. Click **+ ADD** to place the enemy into the scene, then move it onto the correct tile.
6. Adjust the EnemySpawner component for facing, respawn, initial delay and patrol routes as needed.
7. Optionally save the current group as a preset for quick reuse later.

The Enemy Spawner takes you from an empty scene to a populated, loot-bearing dungeon — without writing a single line of code.

---

*CrawlerKIT — Mantis3de*
