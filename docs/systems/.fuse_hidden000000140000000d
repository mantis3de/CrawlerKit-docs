# Setup Guide — Keyhole Door

This guide walks through building a door that requires a specific item (a key) to open. The player picks up the key, holds it on the cursor, clicks the keyhole, and the door opens permanently. The key is consumed and the door stays open through saves and scene reloads.

---

## What you need before you start

- A door prefab with a mesh and an `Animator` component.
- A keyhole mesh (a small slot or ornament on the door frame).
- An `ItemData` ScriptableObject that represents the key. Create one via `Create → CrawlerKit → Item Data` and give it a unique `itemId`, for example `"iron_key"`.

---

## Prefab hierarchy

The setup expects this GameObject structure:

```
Door_Keyhole              ← root object
├── DoorMesh              ← door geometry + Animator
└── Keyhole               ← keyhole geometry + WallKeyhole + BoxCollider
```

Everything wired together (IDs, key assignment, the link from `WallKeyhole` to `TriggerOpenDoor`) is managed by a single `DoorKeyholeSetup` component on the root, so you have one place to look at in the Inspector.

---

## Step 1 — Build the hierarchy

Create an empty GameObject and name it `Door_Keyhole`. This will be the root.

Inside it, create (or drag in) two children:

- **DoorMesh** — your door 3D model. The `Animator` component that drives the open/close animations lives here.
- **Keyhole** — your keyhole 3D model. This child needs a `BoxCollider` so the player's raycast can hit it.

Position the root on a grid cell edge, at the junction between two tiles, oriented so the door faces the correct direction.

---

## Step 2 — Add components to the root

Select the root `Door_Keyhole` and add these four components:

**EdgeBlockerMarker** — registers this door with the grid so the movement system knows it blocks passage. Leave the `Blocker Id` field empty for now; you will fill it in step 5.

**TriggerOpenDoor** — handles the open and close logic, plays the animator triggers, and updates the grid blocker state. Configure the animator fields (see step 4). Leave `Blocker Id` empty for now.

**DoorKeyholeSetup** — the wiring helper. It holds references to the three sibling components and exposes a single custom Inspector that keeps all IDs in sync. Add it last, after the other two are already on the root.

Once `DoorKeyholeSetup` is present, drag the components into its slots in the Inspector:

| Slot | What to drag |
|---|---|
| **Blocker** | `EdgeBlockerMarker` on this root |
| **Door Target** | `TriggerOpenDoor` on this root |
| **Keyhole** | The `Keyhole` child GameObject's `WallKeyhole` component (add it in the next step) |

---

## Step 3 — Add components to the Keyhole child

Select the **Keyhole** child and add:

**WallKeyhole** — the interactive source. When the player clicks the keyhole while holding the correct item, this component consumes the item and activates its targets (the door).

| Field | Value |
|---|---|
| **Keyhole Id** | A short unique string, for example `"door_throne_01"`. This is used to build the Save Id (`keyhole_door_throne_01`). |
| **Required Key Id** | Leave blank — `DoorKeyholeSetup` writes this automatically from the `ItemData` you assign to it. |
| **Key Model** | Optionally drag in a small key mesh child. It becomes visible when the keyhole is used (key inserted). Leave empty if you don't have one. |
| **Used Anim Trigger** | The animator trigger name for the "key inserted" visual, for example `"Used"`. If your keyhole has no animator, leave the default value — it will find no Animator and skip it silently. |

**BoxCollider** — sized to cover the keyhole mesh face. The collider is what the raycast hits when the player clicks. Make sure it isn't too small to click reliably.

!!! note "Interact From Adjacent Cell"
    `WallKeyhole` ignores the **Interact From Adjacent Cell** setting. Keyholes always require the player to be standing on the same cell — you can't insert a key from a distance.

---

## Step 4 — Set up the door Animator

Select **DoorMesh** and open its Animator Controller. You need at minimum four states and two triggers:

```
States
  Idle_Closed     ← default state (door shut)
  Opening         ← plays the open animation
  Idle_Opened     ← door fully open, holds
  Closing         ← plays the close animation

Triggers
  Open            ← sent by TriggerOpenDoor when the door opens
  Close           ← sent by TriggerOpenDoor when the door closes
```

**Transitions:**

- `Idle_Closed → Opening` — condition: trigger `Open` fires. Has Exit Time: off. Transition Duration: 0.
- `Opening → Idle_Opened` — Has Exit Time: on (waits for the animation to finish). Transition Duration: 0.
- `Idle_Opened → Closing` — condition: trigger `Close` fires. Has Exit Time: off. Transition Duration: 0.
- `Closing → Idle_Closed` — Has Exit Time: on. Transition Duration: 0.

Both `Idle_Closed` and `Idle_Opened` are looping hold states (their clips can be a single static frame). `Opening` and `Closing` play once and exit automatically via the Exit Time transitions.

**Reset opposite trigger.** `TriggerOpenDoor` resets the opposite trigger before firing (i.e. it resets `Close` before firing `Open`, and vice versa). This prevents a stale trigger left in the Animator from firing immediately after the next animation completes. You don't need to handle this in the Animator Controller itself.

Now fill in the `TriggerOpenDoor` fields on the root to match:

| Field | Value |
|---|---|
| **Door Animator** | Drag in the `Animator` component from `DoorMesh` |
| **Open Anim Trigger** | `Open` |
| **Close Anim Trigger** | `Close` |
| **Opened State Name** | `Idle_Opened` |
| **Closed State Name** | `Idle_Closed` |
| **Close On Release** | Leave disabled — a keyhole door opens permanently; it has no source that could "release" |
| **Skip Close Guard** | Leave disabled |

---

## Step 5 — Wire everything with DoorKeyholeSetup

Select the root `Door_Keyhole` and look at the **DoorKeyholeSetup** Inspector. It shows a checklist of required components and a **Blocker ID** field at the top.

**Set the Blocker ID.** Type a unique ID such as `door_throne_01`. As you type, `DoorKeyholeSetup` immediately writes that value into `EdgeBlockerMarker.blockerId`, `TriggerOpenDoor.BlockerId` and the `WallKeyhole.sourceId` (`keyhole_door_throne_01`). All three stay in sync from a single field.

**Assign the required key.** Drag your `ItemData` ScriptableObject from the Project window into the **ItemData Asset** slot. The inspector reads `itemData.itemId` and writes it into `WallKeyhole.requiredKeyId` automatically. The field below shows the resolved `itemId` in grey so you can verify it.

**Fix Link button.** If the checklist shows ❌ next to "Keyhole linked to Door", click **Fix root components + link**. This adds `TriggerOpenDoor` to `WallKeyhole`'s **Targets** list so the keyhole knows which door to open.

When all five checklist items show ✅, the door is fully wired:

- ✅ EdgeBlockerMarker
- ✅ TriggerOpenDoor
- ✅ WallKeyhole
- ✅ Collider on Keyhole
- ✅ Keyhole linked to Door

**Export.** In the Dungeon Generator, click **Generate Unique IDs (all doors)** and then **Build Grid (Export JSON)**. The blocker ID you set in step 5 is what the JSON uses to match the physical door to the grid edge.

---

## Step 6 — Test

Press Play. Give the party the correct key item (use the Inventory Editor to add it to a chest or drop it in the scene). Pick up the key so it appears on the cursor. Walk to the door cell and click the keyhole. The key disappears from the cursor, the door opens, and it stays open when you save and reload.

If nothing happens on click, check:

- The player is standing on the exact same cell as the keyhole (keyholes don't use adjacent-cell reach).
- `WallKeyhole.requiredKeyId` matches `ItemData.itemId` exactly — check both in the Inspector.
- The `BoxCollider` on the Keyhole child is large enough to raycast against.
- The **Targets** list on `WallKeyhole` contains the `TriggerOpenDoor` component.

---

## Summary of components

| GameObject | Components |
|---|---|
| `Door_Keyhole` (root) | `EdgeBlockerMarker`, `TriggerOpenDoor`, `DoorKeyholeSetup` |
| `DoorMesh` (child) | `Animator` (with Open/Close triggers and four states) |
| `Keyhole` (child) | `WallKeyhole`, `BoxCollider` |

---

*CrawlerKIT — Mantis3de*
