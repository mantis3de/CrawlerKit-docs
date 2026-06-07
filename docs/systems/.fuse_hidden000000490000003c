# Party Input Handler

`PartyInputHandler` translates **player input into grid movement commands**. It supports keyboard (PC) and UI buttons (mobile/touch), enforces an input cooldown to prevent accidental rapid movement, and exposes a global `MovementLocked` flag that other systems can set to block movement without destroying the component.

---

## Setup

1. Place `PartyInputHandler` on **any GameObject** in the scene — typically the party root or a dedicated input manager.
2. Assign **Party Visuals** in the Inspector (or leave empty — the component finds it automatically).
3. For mobile controls, wire UI buttons to the public `On*Pressed` methods (see [Mobile UI buttons](#mobile-ui-buttons)).

---

## Inspector reference

### Input Settings

| Field | Default | What it does |
|---|---|---|
| **Input Cooldown** | 0.15 s | Minimum time between accepted inputs for each action. Prevents a held key from repeating faster than intended. Each action (forward, backward, etc.) has its own independent cooldown timer. |
| **Block Input During Animation** | ✅ | When enabled, all movement is ignored while `PartyVisuals.IsAnimating` is `true`. This prevents queuing moves faster than the animation can play. |

### References

| Field | What it does |
|---|---|
| **Party Visuals** | Reference used to check `IsAnimating`. Auto-found via `FindFirstObjectByType<PartyVisuals>()` if left empty. |

---

## Keyboard controls (PC)

| Key | Action |
|---|---|
| `W` | Move forward |
| `S` | Move backward |
| `A` | Strafe left |
| `D` | Strafe right |
| `Q` | Turn left (90°) |
| `E` | Turn right (90°) |

---

## Mobile UI buttons

Wire each button's `OnClick` event to the corresponding method on `PartyInputHandler`:

| Method | Action |
|---|---|
| `OnForwardPressed()` | Move forward |
| `OnBackwardPressed()` | Move backward |
| `OnStrafeLeftPressed()` | Strafe left |
| `OnStrafeRightPressed()` | Strafe right |
| `OnTurnLeftPressed()` | Turn left |
| `OnTurnRightPressed()` | Turn right |

UI button methods respect both **Block Input During Animation** and **MovementLocked**, so they behave identically to keyboard input.

---

## MovementLocked

```csharp
PartyInputHandler.MovementLocked = true;   // block all movement
PartyInputHandler.MovementLocked = false;  // restore
```

`MovementLocked` is a **static** flag. Set it from any system that needs to freeze the party — for example `ChestTrigger` sets it when a chest is opened and the player is interacting with the inventory. It blocks `W/A/S/D/Q/E` and all UI button methods. It does **not** block mouse look (`PartyLook`) or UI clicks unrelated to movement.

!!! warning "Remember to unlock"
    Always reset `MovementLocked = false` when your system is done. Because it's static, a stale `true` will freeze the party for the rest of the session.

---

## Reference resolution

On `Start`, and again on every `Update` if `bridge` or `gridSystem` is `null`, `PartyInputHandler` calls `RebindReferences()`:

1. Tries `CrawlerServices.Get<IGridSystem>()` (the Service Locator).
2. Falls back to `GridCoreBridge.Instance`.
3. Falls back to `FindFirstObjectByType<GridCoreBridge>()`.

This means the component **survives scene transitions** and save-load events — it simply re-resolves on the next frame rather than permanently disabling. You can also call `RebindReferences()` manually from a level loader after a transition.

---

## Interaction with other components

| Component | Interaction |
|---|---|
| `PartyVisuals` | Checked for `IsAnimating` each frame (when **Block Input During Animation** is on). |
| `GridCoreBridge` | All moves route through `bridge.MoveForward()` etc., or `gridSystem.TryMove()` if the bridge isn't available. |
| `ChestTrigger` | Sets `MovementLocked = true` on open, `false` on close. Also suppresses the next wall-bounce in `PartyVisuals` when facing a chest. |
| `PartyFallAnimation` | Sets `MovementLocked = true` (via the **Lock Input During Fall** option) for the duration of the fall. |

---

## Troubleshooting

??? failure "Party doesn't move at all"
    Check that `MovementLocked` is `false` — a chest, fall animation, or other system may have locked movement and not unlocked it. Also verify a `GridCoreBridge` exists in the scene.

??? failure "Party moves too fast on key hold"
    Raise **Input Cooldown**. The default (0.15 s) allows roughly 6 steps per second on hold; 0.25 s gives a slower, more deliberate pace.

??? failure "Party can move while an animation is still playing"
    Enable **Block Input During Animation**. If it's already on, check that **Party Visuals** is correctly assigned — if the reference is `null`, `IsAnimating` is never checked.

??? failure "Input stops working after loading a save"
    Call `PartyInputHandler.RebindReferences()` from your save-load callback, or rely on the automatic re-bind that happens in `Update` when `bridge == null`.

---

*CrawlerKIT — Mantis3de*
