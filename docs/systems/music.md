# Music System

The Music System (`CrawlerKitMusic` module) handles background music across all scenes. A single **MusicManager** lives in the Main Menu scene and persists through every scene transition via `DontDestroyOnLoad`. It crossfades between states using dual AudioSources (A/B swap) and automatically switches to Combat music when enemies are alive.

---

## Music states

| State | When it plays |
|---|---|
| `None` | Silence — no track playing |
| `Menu` | Main menu scene |
| `Exploration` | Dungeon levels, no enemies alive |
| `Combat` | Enemies are present in the scene |

Combat detection is **automatic** — the manager listens to `IEnemyManager` events and switches states without any extra code. When the last enemy dies it waits a configurable number of seconds before returning to `Exploration`, so the music doesn't cut in and out during a fight.

---

## Setup

### 1. Create MusicStateData assets

For each state you want music in, create a **MusicStateData** asset:

**Assets → Create → CrawlerKit → Music State Data**

Configure each asset in the Inspector:

| Field | Description |
|---|---|
| **Tracks** | Audio clips to play. Add as many as you want. |
| **Play Order** | `Random` — picks a clip at random (avoids repeating the same one twice in a row). `Sequential` — plays in order, wrapping back to the first. |
| **Loop Track** | If enabled, the current clip loops until the state changes. If disabled, automatically advances to the next clip when it ends. |
| **Volume** | Target volume for this state (0–1). Multiplied by the manager's master volume. |
| **Fade In Time** | Seconds to fade in when entering this state. |
| **Fade Out Time** | Seconds to fade out when leaving this state. |

### 2. Add MusicManager to the Main Menu scene

Create an empty GameObject in your Main Menu scene and add the **Music Manager** component (`CrawlerKit → Music Manager`).

Assign your MusicStateData assets:

| Slot | Asset to assign |
|---|---|
| **Menu Music** | Your menu MusicStateData |
| **Exploration Music** | Your dungeon ambient MusicStateData |
| **Combat Music** | Your combat MusicStateData |

**Inspector settings:**

| Field | Description |
|---|---|
| **Master Volume** | Global volume multiplier (0–1). All state volumes are multiplied by this. |
| **Auto Combat Detection** | When enabled, automatically switches to `Combat` when enemies spawn and back to `Exploration` when they all die. |
| **Combat Exit Delay** | Seconds to wait after the last enemy dies before switching back to Exploration. Prevents rapid switching at the end of a fight. |
| **Debug Log** | Logs every state change to the Console. Useful during development. |

!!! warning "One MusicManager per project"
    Only place the **Music Manager** in the **Main Menu** scene. If the menu scene is loaded again (e.g. "Return to menu"), the duplicate is destroyed automatically. Do **not** add it to every dungeon scene.

### 3. Add SceneMusicSetup to every other scene

Add a **Scene Music Setup** component (`CrawlerKit → Scene Music Setup`) to any GameObject in each scene (e.g. a `_GameManager` object). Set the **State** field to whatever music should play in that scene.

| Field | Description |
|---|---|
| **State** | Music state to activate when this scene loads. |
| **Immediate** | Skip the crossfade — cut to the new track instantly on scene load. |

**Common setups:**

| Scene type | State |
|---|---|
| Main Menu | *(handled by MusicManager directly — no SceneMusicSetup needed)* |
| Dungeon level | `Exploration` |
| Boss arena | `Combat` |
| Cinematic / cutscene | `None` |

---

## Controlling music from code

Access the system through the service locator:

```csharp
var music = CrawlerServices.Get<IMusicSystem>();
```

**Switch state:**
```csharp
music.SetState(MusicState.Combat);
music.SetState(MusicState.Exploration, immediate: true); // no fade
```

**Stop all music:**
```csharp
music.Stop();
music.Stop(immediate: true);
```

**Adjust master volume at runtime:**
```csharp
music.MasterVolume = 0.5f;
```

**Read current state:**
```csharp
if (music.CurrentState == MusicState.Combat)
    Debug.Log("Fight music playing");
```

---

## How crossfading works

The manager keeps two `AudioSource` components (`MusicSource_A` and `MusicSource_B`). When a state change occurs:

1. The **outgoing** source fades to volume 0 over `fadeOutTime` seconds.
2. The **incoming** source starts at volume 0 and fades up to the target volume over `fadeInTime` seconds.
3. Both happen simultaneously — the two fade times are independent.
4. Once the fade completes, the outgoing source is stopped and the roles swap for the next transition.

This means a transition from Exploration to Combat uses the **Combat** state's fade times, and the transition back uses **Exploration**'s fade times.

---

## Architecture

```
CrawlerKitMusic/
└── Runtime/
    ├── IMusicSystem.cs       # Interface + MusicState enum
    ├── MusicStateData.cs     # ScriptableObject — audio clips + settings per state
    ├── MusicManager.cs       # Persistent MonoBehaviour — crossfade, combat detection
    └── SceneMusicSetup.cs    # Per-scene component — declares the base music state
```

The `MusicManager` registers itself as `IMusicSystem` in `CrawlerServices` during `Awake`, so any other module can reach it without a direct reference. It also re-binds to `IEnemyManager` on every scene load, so combat detection works correctly after scene transitions where the enemy manager is recreated.
