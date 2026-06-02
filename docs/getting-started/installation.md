# Installation

This page walks you through installing CrawlerKit into a Unity project and verifying the setup.

## Requirements

| Requirement | Recommended |
| --- | --- |
| Unity | 2021.3 LTS or newer (2022.3 LTS verified) |
| Render pipeline | Universal Render Pipeline (URP) (recommended) |
| UI System | Unity UI (UGUI) |
| Text | TextMeshPro |
| Input | Input System (New Input System) |
| Platform | Any Unity build target (desktop verified) |
| Scripting backend | Mono or IL2CPP |

!!! note "Assembly definitions"
    CrawlerKit ships with assembly definitions (`Mantis3de.CrawlerKit.*`). If your own scripts need to call CrawlerKit at runtime, add a reference to the relevant assembly in your `.asmdef`.

!!! note "Package Manager dependencies"
    CrawlerKit relies on Unity Package Manager packages (URP, UGUI, TextMeshPro, Input System). These are automatically resolved by Unity during import and are not included in the asset.

## Install from the Unity Asset Store

1. Open the **Package Manager** (`Window → Package Manager`).
2. Switch the source dropdown to **My Assets**.
3. Find **CrawlerKit**, click **Download**, then **Import**.
4. Leave all files checked and confirm the import.

## Install from a `.unitypackage`

1. In Unity, choose `Assets → Import Package → Custom Package…`.
2. Select the `CrawlerKit.unitypackage` file.
3. Leave everything checked and click **Import**.

## What gets imported

CrawlerKit installs under `Assets/CrawlerKitFramework/`, organised into modules:

| Module | Purpose |
| --- | --- |
| `CrawlerKitCore` | Service locator, shared types, the CrawlerKit Hub |
| `CrawlerKitParty` | Party data, characters, the Character Generator |
| `CrawlerKitGrid` | Grid system, the Dungeon Generator, level JSON |
| `CrawlerKitInventory` | Items, equipment sets, loot tables, the Inventory Editor |
| `CrawlerKitMagic` | Runes, spells, effects, the Spell Editor |
| `CrawlerKitEnemies` | Enemy data, the Enemy Spawner, spawner runtime |
| `CrawlerKitTriggers` | Doors, traps, buttons and the trigger system |
| `CrawlerKitSaveSystem` | Saving and loading game state |

## Verify the installation

After import, open the Unity top menu. You should see a **CrawlerKit** menu with the editor windows:

- CrawlerKit → Dungeon Generator
- CrawlerKit → Character Generator
- CrawlerKit → Inventory Editor
- CrawlerKit → Spell Editor
- CrawlerKit → Enemy Spawner

If the menu appears, the framework compiled successfully and you are ready to go.

## Open the sample scene

A `SampleScene` is included to show a working setup with a grid, party and UI wired together. Open it from `Assets/CrawlerKitFramework/SampleScene/` and press **Play** to confirm everything runs before you start building your own content.

## Next step

Continue to the [5-minute Quick Start](quick-start.md) to build your first playable room.
