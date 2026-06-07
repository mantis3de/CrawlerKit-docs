# Dungeon Crawler Framework — Spell Editor

## Overview

The Spell Editor is the tool you use to build the entire magic system for your dungeon crawler — runes, spell recipes, effects, VFX and the runtime wiring that connects them all to your scene. You open it from Unity's top menu under **Window → CrawlerKit → Spell Editor**.

The window is laid out in three columns: a **Rune Palette** on the left, a **Spell List** in the centre, and the full **Spell Editor** on the right. In one sentence: you define runes, compose them into recipes, attach an effect, and the editor wires everything to your scene automatically.

The editor needs a **SpellDatabase** asset to work. If it can't find one it shows a single **Create SpellDatabase** button — click it, pick a location, and you're ready.

---

## Rune Palette (left column — step ①)

A rune is one key on the magic input panel. Each rune occupies a fixed cell in a 3×3 grid, so there are nine cells and nine runes maximum. Creation order determines position: rune 1 always goes to (0,0), rune 2 to (1,0), rune 3 to (2,0), rune 4 to (0,1), and so on. That order is mirrored exactly in the runtime RunePanelUI, so what you see in the editor is what the player sees in game.

Each rune appears as an 82×82 tile showing its icon and name; click a tile to select it. Rune types are colour-coded — Fire red, Water blue, Air pale, Earth brown, Death purple, Spirit gold, Balance green, Physical orange, Life light green — and the tile background takes on a desaturated version of that colour so you can tell types apart at a glance.

### Creating a Rune

At the bottom of the palette, **＋ Create New Rune** shows the current count up to the nine-rune cap. Clicking it expands a form with two sprite slots side by side, **ON** and **OFF**. ON is the active sprite shown in the grid when the rune is part of the current recipe; OFF is the inactive sprite shown when the rune is present in the panel but not yet activated by the player.

Drag the ON sprite first — the editor reads its filename and fills in the rune name by stripping the "ON" suffix (so `FireON.png` becomes "Fire"). You can't type the name directly; it always comes from the ON sprite filename, so your sprite names become your rune names. Assign the OFF sprite, pick the **Rune Type** from the dropdown, and click **✔ Create Rune**. The rune is saved under `Data/Runes/` next to your SpellDatabase and appears in the palette immediately.

### Selected Rune Panel

Below the palette grid, the Selected Rune panel shows the clicked rune's icon, name, type and internal ID. You can change the type via a dropdown and reassign the ON and OFF sprite slots (click to open the picker, right-click to clear). A red **🗑 Delete Rune** button removes the rune from every recipe that used it. At the very bottom of the column, **🗑 Clear Rune Palette** deletes all runes at once and strips every recipe clean.

---

## Spell List (centre column — step ②)

Each entry shows a status badge, the spell name, mana cost and cooldown. The badge reflects completion at a glance: ✅ means both recipe and effect are set, ⚠ means one is missing, ✗ means neither is set.

Two buttons sit above the list. **＋ New** creates a blank spell. **⚡ Generate** auto-generates a complete spell — a name, a random recipe, and a Damage effect with random damage and mana cost. Generate needs at least two runes in the palette; it picks a name from a `SpellNames.txt` pool, tries up to 30 times to find a unique rune combination that doesn't conflict with an existing spell, creates both the Spell and Effect assets and adds the spell to the database. If no unique recipe is found after 30 attempts the spell is still created, but with an empty recipe and a warning. A spell count appears at the bottom of the list.

At the very bottom of the list, **🗑 Clear All** permanently deletes every spell, effect, VFX prefab and rune in the project — the SpellDatabase is emptied and all related asset files are removed from disk. A confirmation dialog appears before anything is deleted. Use this when you want to start the magic system from scratch.

---

## Spell Editor (right column — step ③)

Clicking a spell opens it here. At the top is the spell name and a small **🔓 Lock Spell** toggle — when locked, all fields become read-only to prevent accidental edits. The column is divided into four foldout sections: Spell Info, Recipe Editor, Effect and VFX.

### Spell Info

Fields for **Name**, **Level** (the minimum level required to cast), **Energy Cost** (mana) and **Cooldown** in seconds. All are editable directly here, with changes recorded under Undo — no need to open the asset Inspector.

### Recipe Editor

Defines the rune combination that triggers the spell, shown as a fixed 3×3 grid where each cell always corresponds to the same rune (rune 1 top-left, rune 9 bottom-right). Click a cell to toggle its rune ON (shows the ON sprite and the rune's colour) or OFF (shows the OFF sprite, stays dark). Cells for runes that don't exist yet show a grey "—" placeholder.

A conflict checker runs on every click: before adding a rune the editor compares the resulting combination against all other spells. If the pattern already exists, a **Recipe Conflict!** dialog names the spell already using it and the rune is not added. An **Active Runes** list to the right of the grid shows the coordinates and names of active runes plus a count (for example "ON: 3 / 9"), and **Clear All** resets the recipe.

### Effect

Attaches a SpellEffectDefinition to the spell. The **Assigned Effect** slot at the top shows the current effect's type badge, name and editable parameters, or a prompt to pick one from the list below. The **Effects list** shows all SpellEffectDefinition assets, each row with a coloured type badge (DMG, HEAL, DoT, HoT, AoE, BUFF, DEBF), the effect name and its base damage; the assigned effect is highlighted blue with a ✔.

Click the green ＋ on a row to assign it, the red ✕ in the assigned header to unassign, or the red − in a row to delete the asset permanently. Effect parameters are editable in the assigned header: **Effect Type** (a dropdown of seven types), **Damage / Heal** (the base value applied on hit), and — for time-based types only (DoT, HoT, Buff, Debuff) — **Duration** and **Tick Interval**.

A collapsible **ℹ Effect Type Reference** foldout below the list explains every type in the editor:

- **DMG** — instant damage on impact.
- **HEAL** — instant HP restoration.
- **DoT** — Damage over Time; repeated hits at intervals for a set duration (Duration 6 s / Tick 2 s = 3 hits).
- **HoT** — Heal over Time; like DoT but restores HP. Good for regeneration spells.
- **AoE** — Area of Effect; hits all enemies in range at once, applied independently to each target (Duration and Tick are unused for instant AoE).
- **BUFF** — a positive status effect on a friendly target or self; Duration sets how long it lasts, and Damage / Heal can encode its strength (for example +20 attack).
- **DEBF** — Debuff; a negative status effect on an enemy; Duration sets the length and Damage / Heal encodes the penalty (for example −20 defence).

At the bottom of the section, **＋ Create Effect** expands a form for Name, Type, Base DMG, Duration, and Tick Interval (for DoT and HoT). Clicking **✔ Create Effect** saves the asset and assigns it to the current spell immediately.

### VFX & Audio

**VFX** — Three prefab slots: **Cast VFX**, **Projectile** and **Impact VFX**. Each is either an object field with a red ✕ (to clear and delete) when a prefab is assigned, or a cyan **⚡ Create** button when empty. Create generates a ready-made particle system prefab in the `VFX/` folder next to the SpellDatabase — Cast and Impact use a cone-shaped burst, Projectile uses a sphere, and the colour matches the assigned effect's type badge. You can also drag any existing prefab into the object field.

**Audio** — Two AudioClip slots below the VFX fields: **Cast Sound** and **Impact Sound**. Drag any audio asset from your project into the slot. When a clip is assigned, a volume slider (0–1) appears inline so you can tune loudness per-spell without leaving the editor. A red ✕ clears the clip. Cast Sound plays at the caster's position when the spell fires; Impact Sound plays at the target's position when the spell lands (or instantly for non-projectile spells). Volume is respected at runtime through `AudioSource.PlayClipAtPoint`.

The whole section is only active once an effect is assigned; without one it shows "Assign an effect first."

### Validation

A checklist below the VFX section shows three items: **Recipe is valid** (at least one rune in the recipe), **Effect assigned**, and **VFX assigned** (at least one of the three slots filled). VFX is a warning rather than an error — the spell works without it, it just won't look like anything. If a recipe conflict exists at save time an additional orange **Recipe conflict!** message appears.

---

## Save & Sync

At the bottom of the right column, the **💾 Save & Sync** button does several things at once. It writes the current spell and the SpellDatabase to disk, rebuilds the recipe index so the conflict checker stays accurate, and auto-wires the entire magic system to your scene.

Auto-wire specifically: it looks for a **MagicBootstrap** component in the scene and, if there isn't one, creates a `[MagicSystem]` GameObject with the component and links the SpellDatabase to it. It finds all **RunePanelUI** components — in the active scene and in prefabs — and assigns each RuneSlotButton the correct rune in order (Slot 0 = rune 1, Slot 1 = rune 2, and so on), including both ON and OFF sprites. It links RunePanelUI to any sibling **HandSlotUI** components, and if a RunePanelUI has fewer than nine slot buttons it instantiates `RuneSlot_Template` prefabs to fill the gaps. A dialog then confirms what was wired: how many bootstrap components, panel-to-hand links, runes assigned and prefabs updated.

---

## Spell Targeting at Runtime

Hostile spells (Damage, DoT, Debuff) scan the corridor the party is currently facing and hit the first living enemy they find, regardless of how many cells away that enemy stands. The scan stops at closed doors and walls — a spell cannot travel through a locked door to hit an enemy on the other side.

If no enemy is in the corridor, the spell still fires. The projectile flies straight forward and the impact VFX plays against the wall or closed door at the end of the corridor. This matches the behaviour of classic dungeon crawlers like Legend of Grimrock — you never need an enemy present to cast.

Friendly spells (Heal, HoT, Buff) always target the caster or the whole party and are not affected by the corridor scan.

---

## Workflow Summary

1. Open **Window → CrawlerKit → Spell Editor** and create a SpellDatabase if you don't have one.
2. In the Rune Palette, create runes — assign ON and OFF sprites, pick a type, click Create. Repeat up to nine times, one per cell in the 3×3 grid.
3. In the Spell List, click **＋ New** for a blank spell or **⚡ Generate** to auto-generate one.
4. In Spell Info, fill in name, level, energy cost and cooldown.
5. In Recipe Editor, click cells to toggle runes ON; the conflict checker warns if a pattern is already taken.
6. In Effect, create or assign an effect and set its type and parameters.
7. In VFX & Audio, click **⚡ Create** on each VFX slot or drag in your own particle prefabs. Drag audio clips into **Cast Sound** and **Impact Sound** and tune volume with the inline slider.
8. Click **💾 Save & Sync** — the spell is saved and the scene wired automatically.

The Spell Editor takes you from zero to a fully functional magic system — runes, recipes, effects, VFX and runtime wiring — without writing a single line of code.

---

*Dungeon Crawler Framework — Mantis3de*
