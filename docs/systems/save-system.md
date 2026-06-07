# Save System

The `SaveSystem` component writes the party, inventory, enemy and trigger state to JSON files on disk. It saves a **global** file (which scene was active) plus one **per-level** file for every scene the party has visited, so returning to an earlier floor restores exactly how you left it.

---

## Where saves are stored

Save files live inside Unity's per-user persistent data folder, in a dedicated CrawlerKit sub-folder:

```
<Application.persistentDataPath>/CrawlerKit/
    save_global.json          # which scene was last active
    save_<SceneName>.json      # one per visited scene
```

`Application.persistentDataPath` is decided by Unity from your **Project Settings → Player → Company Name / Product Name**. On each platform it resolves to roughly:

| Platform | Location |
|---|---|
| **macOS** | `~/Library/Application Support/<Company>/<Product>/CrawlerKit/` |
| **Windows** | `C:\Users\<user>\AppData\LocalLow\<Company>\<Product>\CrawlerKit\` |
| **Linux** | `~/.config/unity3d/<Company>/<Product>/CrawlerKit/` |

!!! note "Why a sub-folder?"
    The `CrawlerKit/` sub-folder keeps the framework's saves grouped in one place instead of mixing them with whatever else your project writes to `persistentDataPath`. It also means the save location is the same on every machine that imports the asset, regardless of the host project's Company / Product names.

---

## Changing the folder name

The sub-folder name is a single constant in `SaveSystem`:

```csharp
// CrawlerKitSaveSystem/Runtime/SaveSystem.cs
public const string SaveFolderName = "CrawlerKit";
```

Rename it to brand the save location for your own game, or set it to an empty string (`""`) to save directly in `persistentDataPath` with no sub-folder. Every read, write and delete path goes through the shared `SaveSystem.SaveDirectory` property, so changing this one value moves the entire save system consistently — there is no second place to update.

!!! warning "Existing saves don't move"
    Changing `SaveFolderName` (or the project's Company / Product names) changes the path, so the game will no longer find saves written under the old name. During development this is harmless. If you need to keep existing saves, copy the `save_*.json` files into the new folder manually.

---

## API reference

| Member | Description |
|---|---|
| `SaveSystem.SaveFolderName` | `const string` — the sub-folder name. The single place to rebrand the save location. |
| `SaveSystem.SaveDirectory` | `static string` — the full save directory. Created on demand (no-op if it already exists). Used by every save/load/delete path. |
| `SaveSystem.LoadGlobalState()` | `static` — reads `save_global.json` and returns the last active scene. |
| `SaveSystem.HasLevelSave(scene)` | Returns `true` if a save file exists for the given scene. |
| `SaveSystem.DeleteSave()` | Deletes all `save_*.json` files in the save directory. |

---

## Editor behaviour

`SaveSystem` has an **Editor** section with an **Auto Load Save In Editor** toggle (off by default). When off, pressing **Play** in the editor does *not* auto-load the level save, so scene items, enemies and doors stay in their authored state for level design. Standalone builds and the explicit Load / Continue flow are unaffected. Turn it on to test save loading from the editor.
