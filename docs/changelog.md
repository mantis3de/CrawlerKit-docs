# Changelog

All notable changes to CrawlerKit are documented here. The format is based on [Keep a Changelog](https://keepachangelog.com/), and the project aims to follow [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- This MkDocs Material documentation site, with search and GitHub Pages deployment.
- Dedicated documentation pages for all five editors, plus Architecture and API reference.

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
