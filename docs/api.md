# API Reference

Dungeon Crawler Framework is designed to be used through its visual editors, but every system also exposes a clean runtime API so you can extend or integrate it from your own code. This page covers the most useful public entry points.

!!! info "Namespaces"
    Core types live in `Mantis3de.CrawlerKit.Core`; module types live in `Mantis3de.CrawlerKit` and module-specific namespaces. Add an assembly reference to the relevant `Mantis3de.CrawlerKit.*` assembly from your own `.asmdef`.

---

## Service locator — `CrawlerServices`

The static service locator is how modules find each other without hard references. Systems register themselves on startup; everyone else resolves them on demand.

```csharp
public static class CrawlerServices
{
    public static void Register<T>(T service) where T : class;
    public static T    Get<T>()              where T : class;   // null if not registered
    public static bool Has<T>()              where T : class;
    public static void Unregister<T>()       where T : class;
    public static void Clear();
}
```

**Example — resolve the grid and convert a world position to a tile:**

```csharp
using Mantis3de.CrawlerKit.Core;

var grid = CrawlerServices.Get<IGridSystem>();
if (grid != null && grid.IsGridReady)
{
    GridPosition cell = grid.WorldToGrid(transform.position);
    Vector3 worldCenter = grid.GridToWorld(cell);
}
```

**Example — register your own implementation:**

```csharp
public class MyEnemyManager : MonoBehaviour, IEnemyManager
{
    void Awake()  => CrawlerServices.Register<IEnemyManager>(this);
    void OnDestroy() => CrawlerServices.Unregister<IEnemyManager>();
    // ... implement the interface
}
```

---

## Core services

These interfaces are the public contracts each module registers. Resolve them with `CrawlerServices.Get<T>()`.

| Interface | Module | What it does |
| --- | --- | --- |
| `IGridSystem` | Grid | Tile/world conversion, movement queries, grid readiness |
| `IPartySystem` | Party | Party membership, selection, damage/heal/mana |
| `IInventorySystem` | Inventory | Item storage and equipment access |
| `IEnemyManager` | Enemies | Spawning, lookup and death events |
| `IMagicSystem` | Magic | Casting spells |
| `ICombatSystem` | Core/Combat | Resolving party and enemy attacks |
| `ITriggerManager` | Triggers | Firing target IDs (doors, buttons, events) |
| `ISaveSystem` | Save System | Saving, loading and registering saveables |
| `IAbilitySystem` | Core | Generic ability/effect dispatch |

### `IGridSystem` (selected members)

```csharp
public interface IGridSystem
{
    bool IsGridReady { get; }
    Vector3       GridToWorld(GridPosition gridPos);
    GridPosition  WorldToGrid(Vector3 worldPos);
}
```

### `IEnemyManager` (selected members)

```csharp
public interface IEnemyManager
{
    IEnemy GetEnemyAt(GridPosition pos);
    event Action<IEnemy> OnEnemyDied;
}
```

### `IPartySystem` (selected members)

```csharp
public interface IPartySystem
{
    void SelectMember(int slot);
    void DamageMember(int slot, int damage);
    void HealMember(int slot, int amount);
    void RestoreMana(int slot, int amount);
    bool IsPartyAlive();
    void SwapSlots(int slot1, int slot2);
}
```

### `ISaveSystem` and `ISaveable`

Any component can persist state by implementing `ISaveable` and registering with the save system.

```csharp
public interface ISaveSystem
{
    void Register(ISaveable saveable);
    void Unregister(ISaveable saveable);
    void SaveGame(string saveName = "quicksave");
    void LoadGame(string saveName = "quicksave");
    bool HasSave(string saveName = "quicksave");
    void DeleteSave(string saveName);
    void AutoSave();
}

public interface ISaveable
{
    string SaveId { get; }
    object GetSaveData();
    void   LoadSaveData(string json);
}
```

### `IMagicSystem`

```csharp
public interface IMagicSystem
{
    bool TryCast(Spell spell, SpellContext ctx);
    bool CanCast(Spell spell, SpellContext ctx);
}
```

---

## Shared types — `Mantis3de.CrawlerKit.Core`

### `GridPosition`

A 2-D integer grid coordinate with value semantics and operators.

```csharp
public struct GridPosition
{
    public int x;
    public int z;
    public static readonly GridPosition Zero;
    // operator +, -, ==, !=  and Equals are provided
}
```

### `Direction`

`North`, `South`, `East`, `West`. Helper methods on `GridCoreModule` convert and rotate directions:

```csharp
GridPosition off = GridCoreModule.DirectionToOffset(Direction.North);
Direction back    = GridCoreModule.Opposite(Direction.North);
Direction turned  = GridCoreModule.Rotate(Direction.North, clockwise: true);
Direction world   = GridCoreModule.RelativeToWorld(Direction.North, facing);
```

---

## Enemy spawning — `EnemySpawner`

Place this component (or use the [Enemy Spawner](editors/enemy-spawner.md) window) to spawn an enemy at runtime. Its transform position determines the grid tile.

```csharp
[AddComponentMenu("CrawlerKit/Enemy/Enemy Spawner")]
public class EnemySpawner : MonoBehaviour
{
    public EnemyData   enemyData;       // the creature to spawn
    public Direction   startFacing;     // initial facing
    public string      spawnerId;       // unique save id (auto-generated if empty)
    public float       respawnDelay;    // seconds; 0 = no respawn
    public float       initialDelay;    // seconds before first spawn
    public Transform[] waypoints;       // patrol route (PatrolRoute behaviour)
    public PatrolMode  patrolMode;      // Loop or PingPong

    public bool          HasSpawned   { get; }
    public string        InstanceId   { get; }
    public EnemyInstance SpawnedEnemy { get; }
    public void SetSpawnedEnemy(EnemyInstance enemy);
}

public enum PatrolMode { Loop, PingPong }
```

The spawner resolves `IEnemyManager`, `IGridSystem` and the save system through `CrawlerServices` at spawn time, so it never holds direct references to them. An enemy saved as dead is skipped on load, and a saved enemy spawns directly at its stored cell, facing and HP.

---

## Stability note

Public interfaces (the `I*` services), `CrawlerServices`, `GridPosition`/`Direction`, and the editor menu items are the supported, stable surface. Concrete classes and internal helpers may change between versions — prefer the interfaces above when integrating from your own code.
