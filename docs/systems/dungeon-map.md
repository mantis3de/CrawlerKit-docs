# Dungeon Map

The `DungeonMapUI` component draws a fullscreen minimap in the Grimrock style — fog of war, zoom, pan, and a party marker that rotates with the player's facing direction. Press **M** to open and close it. Press **C** while the map is open to re-center on the party.

The map has two rendering modes: **generated** (drawn automatically from the grid data) and **custom PNG** (you paint the map yourself in any graphics program and the system overlays fog of war on top of it at runtime). Both modes support fog of war, secret cells and live door state.

---

## Scene setup

1. On your Canvas, create an empty GameObject and add `DungeonMapUI` to it.
2. Inside that GameObject, create a **Panel** (full-screen, dark background) and assign it to **Map Panel**.
3. Inside the Panel, create a **RawImage** (stretched to fill the panel) and assign it to **Map Image**.
4. Optionally, create an **Image** as a sibling of the RawImage for the party marker sprite and assign it to **Party Icon Image**.

The map panel is hidden on start and toggled by the M key at runtime.

---

## Inspector reference

### References

| Field | Description |
|---|---|
| **Map Panel** | The root panel GameObject. Hidden when the map is closed. |
| **Map Image** | The `RawImage` that receives the generated or custom map texture. |

### Colors (fallback)

These colors are used when no custom sprite is assigned for a given element. All colors can be replaced by sprites (see below).

| Field | Default | Drawn on |
|---|---|---|
| **Floor Color** | Dark blue-grey | Visited floor tiles |
| **Wall Color** | Light grey | Wall edges |
| **Door Color** | Orange-brown | Closed doors |
| **Party Color** | Green | Party dot and facing arrow |
| **Background Color** | Near-black | Area outside the dungeon |
| **Unexplored Color** | Very dark | Tiles the party has not yet visited |
| **Stairs Up Color** | Yellow | Staircase going up |
| **Stairs Down Color** | Cyan | Staircase going down |

### Custom Tile Sprites

Replace any of the color-filled elements with a sprite. The sprite is sampled and stamped onto the map texture at runtime — **Read/Write must be enabled** on the sprite's texture (Texture Import Settings → Advanced → Read/Write Enabled).

| Field | Replaces |
|---|---|
| **Floor Sprite** | Floor Color |
| **Wall Sprite** | Wall Color (rotated for each direction) |
| **Door Closed Sprite** | Door Color (closed state) |
| **Door Open Sprite** | Door Color (open state) |
| **Unexplored Sprite** | Unexplored Color |
| **Stairs Up Sprite** | Stairs Up Color |
| **Stairs Down Sprite** | Stairs Down Color |

### Settings

| Field | Default | Description |
|---|---|---|
| **Tile Pixel Size** | 24 | Pixels per tile in the generated texture. |
| **Wall Thickness** | 2 | Wall edge thickness in pixels. |
| **Min Zoom** | 0.5 | Minimum scroll-wheel zoom level. |
| **Max Zoom** | 3 | Maximum scroll-wheel zoom level. |
| **Zoom Speed** | 0.2 | How fast the scroll wheel zooms. |
| **Target Screen Tile Size** | 40 | UI pixels per tile at zoom = 1. Keeps cell size consistent across maps of different dimensions. |

### Party Icon

| Field | Default | Description |
|---|---|---|
| **Party Icon Image** | none | `Image` component used as the party marker. Rotates automatically with the party's facing. If not assigned, a green dot with a direction arrow is drawn directly into the map texture instead. |
| **Party Icon UI Size** | 32 | Size of the party icon in UI pixels. |

---

## Fog of war

The map reveals tiles progressively as the party explores. Only the tile the party is standing on and its four immediate neighbours are marked as visited. Unvisited tiles are drawn in **Unexplored Color** (or the unexplored sprite) — fully opaque, so the player cannot see wall outlines or room shapes they have not discovered.

**Secret cells** are never drawn on the map at all — not even as unexplored tiles. The player must physically walk into them in the world.

---

## Controls

| Key / Input | Action |
|---|---|
| **M** | Toggle map open / closed |
| **Scroll Wheel** | Zoom in and out (toward the cursor position) |
| **Left Mouse Drag** | Pan the map |
| **C** | Re-center the map on the party |

---

## Custom PNG map

Instead of generating the map programmatically, you can paint it yourself in any graphics program — Photoshop, Aseprite, GIMP, or any pixel editor — and hand it back to the system. At runtime the map loads your image and applies fog of war on top of it.

### Export the base map

1. Enter **Play Mode** in Unity.
2. Press **M** to open the map (this forces the grid to load).
3. In the Inspector, select your `DungeonMapUI` GameObject.
4. Click **Export Map to PNG**.

The file is saved to `Assets/CrawlerKITFramework/Resources/MapExport_{SceneName}.png`. The console logs how many cells and doors were exported.

The exported PNG contains every non-secret cell with its walls and doors drawn at **Export Tile Size** pixels per tile (default 64). A faint helper grid is overlaid on top to make it easier to edit cell by cell. Secret cells are intentionally omitted — they are revealed at runtime when the player walks onto them.

### Map Export settings

| Field | Default | Description |
|---|---|---|
| **Export Tile Size** | 64 | Pixels per tile in the exported PNG. 64 is readable, 128 is comfortable for detailed painting, 256 is very large. |
| **Export Grid Color** | White 15% alpha | Faint grid line overlay. Set alpha to 0 to disable it. |
| **Export Grid Line Thickness** | 1 | Grid line width in pixels. |

### Edit in a graphics program

Open `MapExport_{SceneName}.png` in your editor of choice. The grid lines show you exactly where each cell boundary is. You can:

- Add icons, annotations or labels for rooms and points of interest.
- Paint decorative details — colored regions, patterns, textures.
- Erase or redraw walls to match bespoke geometry.
- Add custom markers for traps, secrets or lore items.

Keep the file at the same pixel dimensions. The system reads it at its original size and maps each cell to the correct screen position.

!!! warning "Fog of war on the custom map"
    Unvisited cells are painted over with **Unexplored Color** at runtime, regardless of what you drew in those pixels. Only cells the party has visited will be revealed. Paint freely — hidden areas will not be visible to the player until discovered.

### Place the edited file back

Save your edited file as `MapExport_{SceneName}.png` and put it in `Assets/CrawlerKITFramework/Resources/`. Unity will pick it up automatically after reimport.

In the Inspector on the texture asset, set **Read/Write Enabled** (Texture Import Settings → Advanced) — the map system needs CPU-readable pixels to apply fog of war on top at runtime.

From that point on, whenever the map is opened the custom PNG is used instead of the generated texture. Fog of war, secret cell reveal and door state updates all continue to work exactly as in generated mode.

### Reverting to generated mode

Delete `MapExport_{SceneName}.png` from Resources. The system falls back to generating the map from grid data.

---

## Secret cells and custom maps

When you export the base PNG, secret cells are skipped — they do not appear in the file, and neither do doors that touch a secret cell. This preserves the mystery: no wall outlines or door shapes give away the existence of a hidden room.

When the player walks into a secret cell at runtime, the cell is marked visited and painted into the map on the next refresh — using the same floor, wall and stair drawing code as the generated mode. The custom PNG is used as a base and the newly-revealed cells are painted on top of it.

---

## How the map knows which PNG to load

The component looks for a file named `MapExport_{SceneName}` in any `Resources` folder at startup. If found, it creates a CPU-readable copy and uses that as the base texture. If not found, it generates the map from the grid. The scene name is `UnityEngine.SceneManagement.SceneManager.GetActiveScene().name`, which matches the filename the export produces.

---

*CrawlerKIT — Mantis3de*
