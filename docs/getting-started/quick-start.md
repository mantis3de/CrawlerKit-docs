# Quick Start

## Getting Started in 5 Minutes

This guide takes you from a fresh import to a playable, first-person dungeon room with a party — using only the visual editors. Each step is roughly a minute.

!!! tip "Before you begin"
    Make sure **Window → CrawlerKit** is visible in Unity's top bar. If it isn't, see [Installation](installation.md).

!!! info "Placeholders"
    All included items, dungeon walls, floors, and props are placeholder assets. Every piece of art can be replaced with your own graphics — swap the prefabs and sprites wherever you see fit. Items are displayed as 2D sprites by default, but can be replaced with 3D models — just swap the 2D sprite for a 3D mesh in the prefab.
Youtube tutorials: 

**Playlist Create Demo Level**:<br>
[Click Link](https://www.youtube.com/playlist?list=PLNQqsD6sx4CQOAk6Y9-8sS7QbkaKWp09J)

**Playlist Editors**:<br>
[Click Link](https://www.youtube.com/playlist?list=PLNQqsD6sx4CQvNYolBlHhJYhk_yk1gtax)

---

### Minute 1 — Create your party

1. Open **Window → CrawlerKit → Character Generator**.
2. On the **Stats** tab, create a few stats (Strength, Dexterity, Vitality, Willpower).
3. On the **Races** tab, create at least one race (e.g. Human).
4. On the **Classes** tab, create at least one class (e.g. Fighter), and set its Primary and Secondary stats.
5. On the **Characters** tab, drag a portrait PNG into the gallery, then click **🎲 Randomize ALL Characters** to fill all four slots instantly.
6. Click **💾 Generate Party Assets**.

!!! tip "Starting fresh or updating an existing party"
    If you already have a party from a previous session, you have two options: click **Clear All** to wipe everything and start from scratch, or simply click **Generate Party Assets** again to overwrite the existing assets in place. The default party characters come from the demo scene — feel free to delete them.

Four `Member_*.asset` files are created in `CrawlerKitParty/Resources/PartyMembers`. The PartyUI loads them automatically at runtime.

→ Full details: [Character Generator](../editors/character-generator.md)

---

### Minute 2 — Generate a dungeon

1. Open **Window → CrawlerKit → Dungeon Generator**.
2. Pick the **TinyArena** preset (a single 10×10 room — perfect for a first test).
3. Assign your **Wall**, **Floor** and **Stairs** prefabs in the **Prefabs** panel. (No prefabs yet? Leave Wall empty to just see the grid.)
4. Click **Generate Flags**, then **Rebuild Dungeon**.
5. Use **Spawn Tools → Party Spawn** to place the party's start point.

→ Full details: [Dungeon Generator](../editors/dungeon-generator.md)

---

### Minute 3 — Author one item and a loot table

1. Open **Window → CrawlerKit → Inventory Editor**.
2. On the **Items** tab, click **+ Create new item**, hit the **Strong** generator button, drag in an icon, and **Save item**.
3. On the **Loot Tables** tab, click **+ Create New Table**, open **Auto-Generate Table**, choose **Enemy**, and click **⚙ Generate Table**.
4. Use **▶ Simulate Roll** to confirm it drops loot, then save.

→ Full details: [Inventory Editor](../editors/inventory-editor.md)

---

### Minute 4 — Place an enemy

1. Open **Window → CrawlerKit → Enemy Spawner**.
2. Click **+** in the Enemies column and assign an enemy prefab.
3. Drag your loot table from the centre column onto the enemy's drop zone.
4. Click **+ ADD** to drop the enemy into the scene, then move it onto a tile.

→ Full details: [Enemy Spawner](../editors/enemy-spawner.md)

---

### Minute 5 — Export and play

1. Back in the **Dungeon Generator**, click **Generate Unique IDs** for doors, traps and walls.
2. Click **Build Grid (Export JSON)** — the level is written to `Resources/GridData_<SceneName>.json`.
3. Find the **MusicManager** in the scene hierarchy and assign your audio clips:
    - **Menu Music** — music played on the main menu.
    - **Exploration Music** — music played while exploring the dungeon.
    - **Combat Music** — music that kicks in during combat.
4. Press **Play** — but see the note below first.

!!! warning "Run from the MainMenu scene"
    Level transitions (stairs, teleporters, scene changes) only work correctly when you start play from the **MainMenu** scene. If you press Play from a dungeon scene directly, transitions will not function. Add `MainMenu` as the first scene in **File → Build Settings** and always use it as your entry point.

You now have a first-person party exploring a dungeon room with an enemy and loot — built entirely without code.

---

## Where to go next

<div class="grid cards" markdown>

-   :material-auto-fix: **Add magic**

    ---

    Build a rune-based spell system in the [Spell Editor](../editors/spell-editor.md).

-   :material-sitemap: **Understand the design**

    ---

    See how it all connects in [Architecture](../architecture.md).

-   :material-code-braces: **Go deeper**

    ---

    Hook into systems from code with the [API Reference](../api.md).

-   :material-help-circle: **Got questions?**

    ---

    Browse the [FAQ](../faq.md).

</div>
