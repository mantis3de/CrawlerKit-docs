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
