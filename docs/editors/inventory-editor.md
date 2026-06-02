# CrawlerKIT — Inventory Editor

## Overview

The Inventory Editor is the tool you use to create every item in your game, bundle items into starter kits for your party, and design what drops as loot. You open it from Unity's top menu under **CrawlerKit → Inventory Editor**.

The window is split into three tabs: **Items**, **Equipment Sets** and **Loot Tables**. Each tab handles a different layer of the inventory system — authoring individual items, dressing your party, and defining drops for enemies and chests.

---

## Tab 1 — Items

The Items tab is the core of the editor. It is split into two panels: a scrollable item list on the left and the full item editor on the right.

### Item List & Filters

The left panel is your item library. A search field at the top filters the list in real time as you type. Below it, category buttons arranged across several rows let you filter by All, Weapons, Shields, Helmets, Armor, Legs, Boots, Cloaks, Amulets, Rings, Consumables, Scrolls, Keys, Quest items and Misc. Each button shows a count of how many items of that type exist, and the active filter is highlighted in blue.

Every row shows the item's icon, its name and a short stat summary (damage for weapons, protection for armor, heal amount for consumables, and so on). A colour-coded power tier tag appears in the top-right corner of each row — Weak in green, Neutral in blue, Strong in orange — and a yellow label below it lists any class restrictions. Clicking a row loads the item into the editor on the right. The **+ Create new item** button sits at the bottom of the list.

### Creating a New Item

Clicking **+ Create new item** shows a blank item on the right. At this point the item exists only in memory — it has not been written to disk. This is deliberate: you fill in every field first and only click **Save item** when you are happy. Clicking **Cancel** discards it and nothing is created.

### Random Generator

A foldout at the very top of the item editor. Three tier buttons — **Weak**, **Neutral** and **Strong** — auto-fill the current item's stats based on its type and the chosen tier. A weak weapon gets low damage, slow speed and poor accuracy; a strong weapon gets high damage, a fast cooldown and one or two stat bonuses. Weak armor gives minimal protection and no penalties; strong armor gives high protection with meaningful evasion and magic penalties. Rings and amulets are always stat items, so even a weak one gets at least one bonus.

A one-line description under the buttons explains what the chosen tier produces. Unsupported types (Keys, Quest items, Misc) show a warning and the buttons won't fire; supported types are Weapon, Shield, Helmet, Armor, Legs, Boots, Cloak, Ring and Amulet. The generator records its result with Undo, so Ctrl+Z restores the previous values.

### Basic Info

A 100×100 drag zone holds the item icon — drag any sprite from the Project window onto it, or drag a Texture2D and the editor resolves the sprite automatically; right-click to clear. Icon changes are buffered locally and only written when you click Save item, so switching items without saving discards the change. Dragging a sprite also pre-fills the item name from the filename, converting snake_case and kebab-case to Title Case (so `iron_sword_01.png` suggests "Iron Sword 01").

To the right are **Item name** and **Item type** (a dropdown that updates the filter counts live), followed by a scrollable **Description** field for lore or tooltip text.

### Visuals

This section assigns extra sprites: **Cursor sprite** (shown while dragging the item in the UI), **World sprite** (used by the generated world prefab to show the item on the dungeon floor) and **Hand sprite** (shown in the character's hand when equipped). Three scale sliders — **Cursor scale**, **World prefab scale** and **Chest scale** — control how large the sprite appears in each context. Every slot works like the icon: drag to assign, right-click to clear, changes stay local until you save.

### Type-Specific Stats

This section changes with the item type:

- **Weapons** — Damage, Attack speed (cooldown in seconds), Hit chance (0–1 slider; DEX adjusts it by ±2% per point at runtime), Protection and STR requirement (a soft threshold — below it the character swings at 70% damage rather than being blocked outright).
- **Shields** — Protection and Evasion penalty (how much the shield restricts movement; a buckler is 0, a tower shield around 0.10).
- **Armor pieces** (Helmet, Armor, Legs, Boots, Cloak, Ring, Amulet) — Protection, Evasion penalty and Magic penalty. These stack across all equipped armor, so high-DEX rogues avoid heavy armor and mages pay for wearing plate.
- **Consumables** — Heal amount, Energy amount and Effect duration.
- **Scrolls** — Energy amount and Effect duration.
- **Keys, Quest items and Misc** — no special stats, just the common fields.

### Stat Bonuses

Add Grimrock-style modifiers that apply while the item is equipped. Click **+ Add stat bonus**, pick a stat from the dropdown of StatDefinition assets, and set the amount — shown as a coloured label (green positive, red negative). Stats already used appear as `(used)` so you don't assign the same one twice, and the red ✕ removes a row. If no StatDefinitions exist yet the section shows a warning and a path to create them, and a small **↻ Refresh stats** button picks up newly created stats without restarting the editor.

### Class Restrictions

Controls which classes can equip the item. Currently allowed classes appear as blue pill labels, each with an ✕ to remove. Click **+ Add Class ▾** to add one from the dropdown of ClassData assets; classes already in the list are disabled so you can't duplicate them. An empty list means anyone can equip the item — that is the default.

### Common Fields

At the bottom: **Stacking & weight** (a Stackable toggle, a Max stack field that is disabled when not stackable, and Weight), **Economy** (a Sellable toggle and a Sell price field disabled when not sellable), and **Rarity** (an integer slider 0–2 mapping to the Weak/Neutral/Strong tier tag in the list).

### Saving & Bottom Buttons

- **Cancel** — discards unsaved changes; asks whether to discard an unsaved item or revert one already on disk.
- **Generate World Prefab** (green) — creates a prefab with a WorldItem component, a flat SpriteRenderer using the world sprite, and a BoxCollider sized to match. It is saved next to the item asset and linked to the `worldPrefab` field. Disabled until the item is saved.
- **Generate Alcove Prefab** (blue) — creates a WallAlcove prefab pre-configured with the item's ID, the world prefab as its visual, and a BoxCollider for interaction raycasts. Drop it into a wall, adjust the collider, wire the triggers and you have a working item puzzle. Disabled until a world prefab exists.
- **Save item ★** (yellow when unsaved) — on first save a file dialog lets you choose where the `.asset` goes, and the item ID is derived from the filename; later saves just write changes. The inventory database syncs automatically after every save.
- **Delete** (red) — permanently deletes the asset file; disabled for unsaved items.

---

## Tab 2 — Equipment Sets

An equipment set is a bundle of items you can pre-assign to a party member as their starting gear. The tab has three panels: a set list on the left, an item browser in the middle, and a set editor on the right.

### Set List

Shows all EquipmentSetData assets in the project. Click an entry to open it. The ⠿ handle on the left of each row lets you drag a set straight onto a party member portrait in the editor. The red X deletes the set, and **+ Create New Set** at the bottom opens a save dialog in the default EquipmentSets folder.

### Item Browser

The middle panel lists every item in the project with the same search and type filters as Tab 1. Drag an item from the browser onto a slot in the set editor to populate it; drop targets highlight blue on hover.

### Set Editor

Shows the set's **Set name** and **Description**, then two groups of slots. **Body Slots** are Head, Chest, Cloak, Legs, Feet, Amulet and Ring; **Hands** are Left Hand and Right Hand. Each slot is a 72×72 icon box showing the item icon and a truncated name when filled, or a placeholder letter when empty. Drag an item onto a slot to assign it, right-click to clear, left-click to ping the asset in the Project window.

Below the slots, the **Party** section shows your four party members (FrontLeft, FrontRight, BackLeft, BackRight) with names and classes. Drag the selected set from the list onto a portrait to assign it. If the set contains items restricted to other classes the portrait turns red and a dialog asks whether to assign anyway; otherwise it turns blue. After assigning, the portrait shows the set name in orange, with a red ✕ to clear it. **Save Set** writes the set to disk and pushes the assignments to the Character Generator, refreshing it live if it is open.

---

## Tab 3 — Loot Tables

A loot table is a ScriptableObject defining what drops from an enemy, chest or any other source at runtime. The tab uses the same three-panel layout: table list, item browser and table editor.

### Loot Table Editor

The editor shows the table's **Identity** (name and description) and a **Roll Settings** section with three modes:

- **Pick One** — one item per roll, chosen by weighted random. Weight 10 versus Weight 1 means roughly 10× the drop frequency. Best for doors, small pots and single guaranteed drops.
- **Pick N** — like Pick One but draws several items; you set Min Rolls and Max Rolls, and the actual count is random in that range. Best for treasure chests.
- **Roll All** — every entry rolls independently, with weight meaning a drop chance in percent (1–100). Multiple items, or none, can drop per roll. Best for enemy death loot and breakable barrels.

The **Drop Chance** slider is a master gate: at 1.0 the table always fires, at 0.6 there's a 40% chance the whole table drops nothing. A warning label shows the empty-drop percentage when it's below 100%. A stats bar below the settings reports entry count, total weight and how many items are picked per roll.

### Loot Table Entries

Each entry is an item slot plus weight and a quantity range. The left side is a 64×64 icon drop target (drag from the browser, right-click to clear, left-click to ping). The right side shows the item name, a Weight field (Chance% in Roll All mode) and the resulting drop probability as a colour-coded percentage — green above 50%, yellow above 20%, red below. **Min** and **Max** quantity fields set how many of the item can drop per roll. **+ Add Entry** appends a new entry and the red Remove button deletes a row.

### Auto-Generate Table

A foldout below the entries with three context presets:

- **Enemy** — Roll All mode, 30–60% drop chance, low equipment chance, higher consumables.
- **Chest** — Pick One mode, 100% drop chance, a mix of equipment and consumables with tiered weights.
- **Boss** — Pick N (2–4 rolls), 100% drop chance, guaranteed better loot and a higher equipment chance.

Then choose the item tier (Weak, Neutral, Strong) and the number of entries (2–10), and click **⚙ Generate Table** to fill it with random matching items from your project. The operation is recorded with Undo.

### Loot Simulator

A foldout to test a table without running the game. **▶ Simulate Roll** rolls the table once using the exact runtime logic and lists the resulting items and quantities, or "Nothing dropped" if Drop Chance withheld everything. A log line below shows the raw roll details.

---

## Workflow Summary

**Creating an item:** create a new item → optionally hit a Random Generator tier → drag in icon and visual sprites → fill in name, type, description, stats, bonuses and restrictions → Save item → Generate World Prefab → optionally Generate Alcove Prefab.

**Setting up party gear:** open Equipment Sets → Create New Set → drag items onto the body and hand slots → drag the set onto a party portrait → Save Set.

**Creating a loot table:** open Loot Tables → Create New Table → pick a roll mode and Drop Chance → use Auto-Generate Table or add entries manually → test with the Loot Simulator → save.

The Inventory Editor takes you from a blank item asset to a fully configured loot economy without writing a single line of code.

---

*CrawlerKIT — Mantis3de*
