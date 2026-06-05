# Changelog

All notable changes to CrawlerKit are documented here. The format is based on [Keep a Changelog](https://keepachangelog.com/), and the project aims to follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- **ProjectileTrap Aim Target** — `ProjectileTrap` has a new optional **Aim Target** field. Drag a `Transform` from the `PartyVisuals` prefab (e.g. a child at chest height named `AimTarget`) into this slot and projectiles will arc vertically toward that point instead of traveling flat. Leave empty for the original flat, direction-only shot.
- **Open pit documentation** — documented the open pit setup (visible walkable hole, instant kill) as a distinct variant of `TrapDoorTrap` with `TrapDoor` cell type. Clarified the difference between **Pit** (impassable/decorative, no component needed) and **TrapDoor** (walkable, trap fires on enter).

### Added (previous)
- This MkDocs Material documentation site, with search and GitHub Pages deployment.
- Dedicated documentation pages for all five editors, plus Architecture and API reference.
- **Spell audio** — `SpellEffectDefinition` now has `castSound`, `impactSound`, `castVolume` and `impactVolume` fields. Sounds play at the correct world-space position via `AudioSource.PlayClipAtPoint`. Both slots are exposed directly in the Spell Editor under the new **VFX & Audio** section so no Inspector digging is needed.
- **HealthBarAnchor component** — place this on an empty child GameObject in the enemy prefab to control the HP bar position visually in the Scene view. The red bar gizmo shows exactly where the bar will float. Replaces the `healthBarYOffset` and `healthBarZOffset` fields that were previously on EnemyData.
- **VFXAimPoint component** — place this on an empty child GameObject in the enemy prefab at chest height. Spell projectiles and impact VFX aim at this transform's world position. The cyan sphere and crosshair gizmo make placement easy in the Scene view. Replaces the `vfxAimYOffset` field that was previously on EnemyData.
- **Corridor spell targeting** — hostile spells now scan the entire corridor in the party's facing direction and target the first living enemy found, not just the enemy in the adjacent cell. The scan respects closed doors and walls — spells cannot pass through them.
- **Casting into empty corridors** — hostile spells can be cast even when no enemy is present. The projectile flies straight forward and the impact VFX plays at the wall or closed door at the end of the corridor, matching the feel of classic dungeon crawlers.

- **Weapon and item audio** — `ItemData` has a new Audio section with `Attack Sound` (weapon swing), `Equip Sound`, `Pickup Sound`, `Drop Sound` and a shared `Item Sound Volume` slider. `HandSlotUI` adds `Default Swing Sound` (fallback when a weapon has no attack sound) and `Miss Sound`. All item sounds play at the camera position (2D).
- **Footstep and party-hit audio** — `PartyVisuals` now has a `Footstep Sounds` array (randomly chosen on every step) and a `Party Hit Sound` played whenever an enemy's attack lands. Both play through a 2D AudioSource on the PartyVisuals GameObject.
- **Pickup audio** — `WorldItem.TryPickUp()` plays the item's `Pickup Sound` at the item's world position when the party successfully picks it up.
- **Trigger audio** — `TriggerSource` base class now has `Interact Sound` and `Interact Volume` fields. All interactive sources (WallButton, WallLever, WallKeyhole and any custom source) play this sound automatically on successful interaction. `TriggerOpenDoor` has separate `Open Sound`, `Close Sound` and `Door Sound Volume` fields that play at the door's world-space position (3D spatial audio).

### Changed
- `EnemyData` — removed `healthBarYOffset`, `healthBarZOffset`, and `vfxAimYOffset`. Position the HP bar and VFX aim point using the new prefab child components instead.
- Spell Editor section renamed from **4. VFX** to **4. VFX & Audio**.
- `RunePanelUI` error message when no enemy is in range changed from "No enemy in front!" to "No enemy in range!" to reflect the broader corridor scan.

---

## [1.0.0]

First public release of CrawlerKit.

### Added
- **Character Generator** — visual party builder with Stats, Races, Classes and Characters tabs; portrait import, four stat-roll modes (Balanced, Class, Heroic, Cursed), starter equipment, permadeath, and one-click party asset generation.
- **Dungeon Generator** — procedural generation with BSP and Maze algorithms, ready-made presets, manual tile editing, layout snapshots, door/trap/secret-wall/chest libraries, unique-ID generation, and JSON export.
- **Inventory Editor** — item authoring with a random stat generator, equipment sets, and weighted loot tables (Pick One / Pick N / Roll All) with a built-in loot simulator and auto-generate presets.
- **Spell Editor** — rune palette (up to a 3×3 grid), recipe editor with conflict checking, seven effect types, VFX generation, and one-click Save & Sync that auto-wires the magic system to the scene.
- **Enemy Spawner** — three-column window for assembling enemies, browsing and assigning loot tables by drag-and-drop, placing enemies into the scene, and saving reusable encounter presets; runtime support for respawn, initial delay and patrol routes.
- **Core framework** — `CrawlerServices` service locator, shared grid types (`GridPosition`, `Direction`), and module interfaces for grid, party, inventory, magic, enemies, triggers and saving.
- **Save System** — persistence of party, enemy and grid state via the `ISaveable` / `ISaveSystem` contract.
- **Sample scene** demonstrating a fully wired setup.

---

!!! note "Version numbers"
    Replace the placeholder version above with your actual Asset Store release version when you publish. Add a new dated section under **[Unreleased]** for each update so customers can track changes.
