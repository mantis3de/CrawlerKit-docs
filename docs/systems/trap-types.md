# Trap Types — Klasyczne pułapki lochów

Ten przewodnik opisuje 10 gotowych konfiguracji pułapek zbudowanych z `TrapDoorTrap` i `ProjectileTrap`. Każdy typ wyjaśnia co widzi gracz, co się dzieje, jak zbudować prefab i jak ustawić każde pole w Inspectorze.

!!! note "Przed lekturą"
    Zanim zaczniesz od któregokolwiek typu, przeczytaj [Setup Guide — Pit & Trapdoor](setup-trapdoor.md) — tam znajdziesz wyjaśnienie hierarchii prefabu, TrapMarkera i Animatora krok po kroku.

---

## Type 1 — Open Pit (Otwarty dół)

**Co się dzieje:** Gracz widzi dziurę w podłodze. Wchodzi i natychmiast ginie.

Najprostsza możliwa pułapka. Żadnego komponentu `TrapDoorTrap` — geometria i typ celi robią całą robotę.

**Setup:**

1. W Dungeon Generatorze oznacz celę jako **TrapDoor**.
2. Postaw mesh dołu (dziura, ściany z każdej strony, ściany biegnące w dół).
3. Żadnych komponentów nie dodawaj — gotowe.

Drużyna może wejść na celę i natychmiast spada. Śmierć jest błyskawiczna, bez animacji, bez dźwięku (chyba że podepniesz coś pod **On Fall Triggered** — ale przy tym typie i tak nie masz `TrapDoorTrap`, więc podepnij to przez UnityEvent na innym obiekcie).

---

## Type 2 — Timed Trapdoor (Klapa z opóźnieniem)

**Co się dzieje:** Gracz wchodzi na kafel, który wygląda jak zwykła podłoga. Po chwili klapa odpada pod jego nogami i wpada w dół.

Klasyczna ukryta pułapka — gracz nie ma pojęcia że to dziura, dopóki nie jest za późno.

### Hierarchia prefabu

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← zamknięta klapa (wygląda jak normalny kafel) + Animator
```

### Animator — stany i przejścia

Stwórz Animator Controller na `TrapMesh` z następującymi stanami:

```
Idle_Closed    ← domyślny (Set as Layer Default State) — klapa zamknięta
Opening        ← animacja otwierania
Idle_Open      ← klapa otwarta, widać dziurę
```

W zakładce **Parameters** dodaj jeden trigger:

| Nazwa | Typ |
|---|---|
| `Open` | Trigger |

Przejścia:

| Od | Do | Warunek | Has Exit Time |
|---|---|---|---|
| `Idle_Closed` | `Opening` | trigger `Open` | ❌ |
| `Opening` | `Idle_Open` | *(brak — po skończeniu klipu)* | ✅ |

### Inspector — TrapDoorTrap

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.8–1.5 s *(im dłużej, tym bardziej okrutna pułapka — gracz ma czas zdać sobie sprawę, ale za mało żeby uciec)* |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |
| **Trap Animator** | Animator z TrapMesh |
| **Open Trigger** | `Open` |
| **Close Trigger** | *(zostaw puste — klapa nigdy się nie zamyka)* |

Levera nie potrzebujesz. Brak zewnętrznych połączeń.

!!! tip
    Im dłuższy **Trigger Delay**, tym bardziej sadystyczna pułapka — gracz słyszy skrzypienie klapy i wie co się dzieje, ale nie zdąży się cofnąć.

---

## Type 3 — Lever-Controlled Trapdoor (Klapa sterowana leverem)

**Co się dzieje:** Gracz stoi na klapie. Ktoś (albo sam gracz) pociąga lever gdzieś indziej — klapa otwiera się pod nim i wpada w dół. Drugie pociągnięcie levera zamyka klapę z powrotem.

Klasyczna pułapka zaufania — pociągnij lever i obserwuj co się dzieje.

### Hierarchia prefabu

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← zamknięta klapa + Animator

[gdzieś na ścianie]
WallLever             ← WallLever (TriggerSource)
```

### Animator — stany i przejścia

Stwórz Animator Controller na `TrapMesh`:

```
Idle_Closed    ← domyślny stan
Opening        ← animacja otwierania
Idle_Open      ← klapa otwarta
Closing        ← animacja zamykania
```

W zakładce **Parameters**:

| Nazwa | Typ |
|---|---|
| `Open` | Trigger |
| `Close` | Trigger |

Przejścia:

| Od | Do | Warunek | Has Exit Time |
|---|---|---|---|
| `Idle_Closed` | `Opening` | trigger `Open` | ❌ |
| `Opening` | `Idle_Open` | *(brak — po skończeniu klipu)* | ✅ |
| `Idle_Open` | `Closing` | trigger `Close` | ❌ |
| `Closing` | `Idle_Closed` | *(brak — po skończeniu klipu)* | ✅ |

### Inspector — TrapDoorTrap

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ *(lever działa tylko gdy drużyna stoi na klapie)* |
| **Trigger Delay** | 0.2 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trap Animator** | Animator z TrapMesh |
| **Open Trigger** | `Open` |
| **Close Trigger** | `Close` |

### Podłączenie levera

1. Zaznacz GameObject z `WallLever`.
2. W Inspektorze znajdź listę **Targets**.
3. Kliknij **+** i przeciągnij GameObject z `TrapDoorTrap` do slotu.

Pierwsze pociągnięcie → lever aktywowany → `OnTriggered` → klapa otwarta (`Open`).
Drugie pociągnięcie → lever deaktywowany → `OnReleased` → klapa zamknięta (`Close`).

!!! tip
    Ustaw lever w tym samym pomieszczeniu co klapa — gracz musi podjąć decyzję: stanąć na klapie i pociągnąć, czy najpierw zbadać sytuację.

---

## Type 4 — Lever Puzzle (Przełączane klapy)

**Co się dzieje:** Kilka klap blokuje przejście. Jeden lever przełącza ich stan — każde pociągnięcie otwiera/zamyka inną kombinację. Gracz musi znaleźć kombinację, w której wszystkie klapy na ścieżce są zamknięte i można przejść.

### Setup

Użyj jednego `WallLever` (toggle). Połącz kilka komponentów `TrapDoorTrap` jako **Targets** na leverze.

Każdej klapie ustaw inny **Trigger Delay** — dzięki temu nie otwierają się równocześnie, tylko jedna po drugiej, tworząc przesuwające się „okno" bezpiecznych kafli:

| Klapa | Auto Trigger | Require Party | Trigger Delay | Permanent | Trigger Once |
|---|---|---|---|---|---|
| Klapa A | ❌ | ❌ | 0 s | ❌ | ❌ |
| Klapa B | ❌ | ❌ | 0.3 s | ❌ | ❌ |
| Klapa C | ❌ | ❌ | 0.6 s | ❌ | ❌ |

Lever aktywowany → klapy otwierają się po kolei. Lever deaktywowany → zamykają się. Gracz przełącza lever wielokrotnie i obserwuje rytm, żeby znaleźć bezpieczną chwilę na przejście.

!!! note
    **Permanent = false** i **Trigger Once = false** na każdej klapie — bez tego zablokują się na otwarto po pierwszym wyzwoleniu i nie będą się resetować.

### Animator

Każda klapa potrzebuje pełnego zestawu stanów jak w Type 3 (`Idle_Closed`, `Opening`, `Idle_Open`, `Closing`) i obu triggerów (`Open`, `Close`).

---

## Type 5 — Logic Gate Puzzle (Zamek wielodźwigniowy)

**Co się dzieje:** Trzy levery sterują jedną lub kilkoma klapami. Klapa otwiera się tylko gdy wszystkie trzy levery są w złej pozycji — gracz musi znaleźć jedyną bezpieczną kombinację.

### Setup

1. Stwórz pusty GameObject, dodaj komponent `TriggerLogicGate` (typ: **AND**).
2. Połącz wszystkie trzy `WallLever` jako wejścia `TriggerLogicGate`.
3. Połącz wyjście `TriggerLogicGate` z `TrapDoorTrap`.

Klapa otwiera się gdy wszystkie levery są aktywne jednocześnie. Każda inna kombinacja = bezpiecznie.

Możesz odwrócić logikę bramką **NOT** lub **NOR** — klapa zaczyna otwarta i zamyka się dopiero gdy gracz znajdzie właściwą kombinację.

### Inspector — TrapDoorTrap

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Permanent** | ❌ |
| **Trigger Once** | ❌ |

---

## Type 6 — Pressure Plate Trap (Płyta naciskowa)

**Co się dzieje:** Gracz staje na płycie naciskowej — gdzieś indziej klapa się otwiera. Kiedy gracz schodzi z płyty, klapa zamyka się. Klasyczny setup „trzymaj płytę żeby przejść" — ale z twistem: płyta otwiera klapę pod innym graczem.

### Setup

Podłącz `PressurePlate` → **Targets** → `TrapDoorTrap`.

### Inspector — TrapDoorTrap

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Trigger Delay** | 0 s |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |

### Animator

Pełny zestaw stanów jak w Type 3 + oba triggery `Open` i `Close`.

Gracz staje na płycie → klapa otwarta. Następnym razem gdy ktoś przejdzie przez klapę → wpada.

---

## Type 7 — Repeating Pit (Pułapka z resetem)

**Co się dzieje:** Klapa otwiera się pod graczem, ten wpada i ginie. Kilka sekund później klapa zamyka się i resetuje — czeka na następną ofiarę.

Przydatne w korytarzach, przez które gracz musi przechodzić wielokrotnie (backtracking, respawny).

### Hierarchia prefabu

```
TrapDoor              ← root: TrapMarker + TrapDoorTrap
└── TrapMesh          ← Animator z pełnym zestawem stanów (Idle_Closed / Opening / Idle_Open / Closing)
```

### Animator

Identyczny jak w Type 3 — wszystkie cztery stany i oba triggery (`Open`, `Close`).

### Inspector — TrapDoorTrap

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ❌ |
| **Close Delay** | 3 s |
| **Trigger Once** | ❌ |
| **Trap Animator** | Animator z TrapMesh |
| **Open Trigger** | `Open` |
| **Close Trigger** | `Close` |

Sekwencja: drużyna wchodzi → po 0.3 s klapa się otwiera → drużyna wpada → po 3 s klapa się zamyka → reset.

---

## Type 8 — Crossbow Corridor (Korytarz bełtów)

**Co się dzieje:** Gracz wchodzi w korytarz, wdepnie na płytę naciskową i ze ściany wylatuje strzała/bełt.

Klasyczny korytarz z grotami — żadnej klapy, tylko `ProjectileTrap` + `PressurePlate`.

### Setup

`PressurePlate` → **Targets** → `ProjectileTrap`.

### Inspector — ProjectileTrap

| Pole | Wartość |
|---|---|
| **Projectile Prefab** | prefab strzały / bełtu |
| **Fire Direction** | prostopadle do kierunku ruchu gracza |
| **Damage** | 20–30 |
| **Aim Target** | Transform z `PartyVisuals` |

Postaw kilka `ProjectileTrap` po obu stronach korytarza z przeciwnymi wartościami **Fire Direction** — strzały lecą jednocześnie z lewej i prawej.

---

## Type 9 — Pit + Crossbow Combo

**Co się dzieje:** Gracz wchodzi na celę z klapą. Klapa otwiera się pod nim. W tym samym momencie strzała leci ze ściany. Nawet jeśli klapa jest sterowana leverem i gracz nie stoi na niej — strzała i tak go trafia.

`TrapDoorTrap` i `ProjectileTrap` wyzwalane z tego samego źródła.

### Wariant A — wspólny lever

Jeden `WallLever` → **Targets**: oba `TrapDoorTrap` i `ProjectileTrap` jednocześnie.

### Wariant B — strzała odpala się po otwarciu klapy

Zostaw lever. Użyj eventu **On Fall Triggered** na `TrapDoorTrap` żeby wywołać `ProjectileTrap.Trigger()` przez UnityEvent. Strzała leci dopiero po otwarciu klapy.

### Inspector — TrapDoorTrap

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **On Fall Triggered** | → `ProjectileTrap.Trigger()` |

### Inspector — ProjectileTrap

| Pole | Wartość |
|---|---|
| **Fire Direction** | ze ściany bocznej |
| **Damage** | 25 |

---

## Type 10 — Gauntlet (Korytarz śmierci)

**Co się dzieje:** Gracz musi przejść przez korytarz pełen klap i strzał. Klapy otwierają się kolejno (każda z innym opóźnieniem), strzały lecą z płyty na wejściu. Istnieje bezpieczna ścieżka — gracz musi ją znaleźć.

Połączenie Type 4 (cykliczne klapy), Type 7 (reset), Type 8 (strzały).

### Setup

1. Postaw 4–6 celi `TrapDoor` wzdłuż korytarza. Każda z innym **Trigger Delay** (0 s, 0.5 s, 1 s, 1.5 s…), **Permanent = false**, **Trigger Once = false**.
2. Podłącz jeden `WallLever` do wszystkich klap — przełączanie levera zmienia które klapy są aktualnie otwarte.
3. Postaw `PressurePlate` na wejściu do korytarza, podłączone do `ProjectileTrap` po obu stronach.
4. Gracz najpierw obserwuje rytm klap, potem wchodzi i biegnie przez okno bezpiecznego przejścia.

### Inspector — każda klapa

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Permanent** | ❌ |
| **Trigger Once** | ❌ |
| **Close Delay** | 2–3 s |
| **Trigger Delay** | inny dla każdej klapy (0, 0.5, 1.0, 1.5…) |

### Animator — każda klapa

Pełny zestaw stanów jak w Type 3 (`Idle_Closed`, `Opening`, `Idle_Open`, `Closing`) + triggery `Open` i `Close`.

!!! tip
    Dodaj **On Fall Triggered** → `TriggerPlaySound` na każdej klapie — dźwięk wpadającej drużyny słyszany z końca korytarza buduje napięcie przed kolejną próbą.

---

*CrawlerKIT — Mantis3de*
