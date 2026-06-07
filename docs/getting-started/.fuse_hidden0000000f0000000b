# Installation

This page walks you through installing CrawlerKit into a Unity project and verifying the setup.

## Requirements

| Requirement | Recommended |
| --- | --- |
| Unity | 2022.3 LTS or newer (2022.3 LTS verified) |
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

## Create a URP project

CrawlerKit is built for the **Universal Render Pipeline**. Before importing the asset, make sure your Unity project uses URP:

- When creating a new project in Unity Hub, choose the **3D (URP)** template.
- If your project already exists but uses the Built-in pipeline, you will need to convert it — see Unity's official [URP migration guide](https://docs.unity3d.com/Packages/com.unity.render-pipelines.universal@latest/index.html) before continuing.

Once the project is open, create an empty scene (`File → New Scene → Basic (URP)`) and save it as your starting point. This ensures the camera and lighting are configured correctly for URP before you wire up CrawlerKit.

## Install the Input System package

CrawlerKit requires Unity's **New Input System** package. If it is not already installed:

1. Open **Window → Package Manager**.
2. Switch the source dropdown to **Unity Registry**.
3. Search for **Input System**, select it and click **Install**.
4. Unity will ask to enable the new backends — click **Yes**. The editor will restart.

After the restart, open **Edit → Project Settings → Player → Other Settings** and find **Active Input Handling**. Set it to **Both** so that legacy `UnityEngine.Input` calls (used by some Unity UI components) continue to work alongside the new Input System.

!!! warning "Active Input Handling"
    If you leave this set to **Input System Package (New)** only, certain Unity UI drag-and-drop interactions may stop working. **Both** is the recommended setting for CrawlerKit projects.

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

After import, open **Window** in Unity's top menu. You should see a **CrawlerKit** submenu with the editor windows:

- Window → CrawlerKit → Dungeon Generator
- Window → CrawlerKit → Character Generator
- Window → CrawlerKit → Inventory Editor
- Window → CrawlerKit → Spell Editor
- Window → CrawlerKit → Enemy Spawner

If the menu appears, the framework compiled successfully and you are ready to go.

## Add scenes to Build Settings

Level transitions between scenes only work if Unity knows about all the scenes. After importing CrawlerKit:

1. Open **File → Build Settings**.
2. Click **Add Open Scenes** or drag the following scenes from the Project window into the **Scenes In Build** list, in this order:
    - `Assets/CrawlerKitFramework/Scenes/MainMenu`
    - `Assets/CrawlerKitFramework/Scenes/Level_01`
    - `Assets/CrawlerKitFramework/Scenes/Level_02`
3. Close Build Settings.

!!! warning "MainMenu must be index 0"
    Unity loads scenes by their build index. Make sure **MainMenu** is at the top of the list (index 0) — this is the required entry point. Starting play from any other scene may result in missing references or broken transitions.

## Open the sample scene

To try the demo, open `Assets/CrawlerKitFramework/Scenes/MainMenu` and press **Play**. All three scenes must be in Build Settings (see above) for level transitions to work correctly.

## Expected console warnings after import

When you first press Play, the Console will show warnings from `MusicManager` like:

```
[MusicManager] State 'Menu' has no valid AudioClip assigned in MusicStateData.
[MusicManager] State 'Exploration' has no valid AudioClip assigned in MusicStateData.
[MusicManager] State 'Combat' has no valid AudioClip assigned in MusicStateData.
```

This is normal — no music clips are assigned yet. The warning fires when a `MusicStateData` slot is assigned but contains no `AudioClip`. To silence the warnings, assign your own audio clips to **Menu Music**, **Exploration Music**, and **Combat Music** on the `MusicManager` in the MainMenu scene (see [Quick Start — Minute 5](quick-start.md#minute-5-export-and-play)). Setting a slot to **None** entirely will also stop the warning for that state — the system will simply play nothing.

## Next step

Continue to the [5-minute Quick Start](quick-start.md) to build your first playable room.
