# Trap Types — Classic Dungeon Traps

Ten przewodnik pokazuje 10 gotowych do użycia konfiguracji pułapek zbudowanych z `TrapDoorTrap` i `ProjectileTrap`. Każdy typ opisuje co gracz widzi, co się dzieje, i jak to ustawić w Inspektorze.

---

## Type 1 — Open Pit

**Co się dzieje:** Gracz widzi dziurę w podłodze. Wchodzi i natychmiast ginie.

Najprostszy możliwy typ. Brak komponentu `TrapDoorTrap` — sama geometria i cell type robią robotę.

**Setup:**

1. W Dungeon Generatorze oznacz komórkę jako **TrapDoor**.
2. Umieść pit mesh (dziura, ściany dookoła, ściany w dół).
3. Nie dodawaj żadnego komponentu — koniec.

Gracz może wejść w komórkę. Śmierć natychmiastowa, brak animacji, brak dźwięku (chyba że podepniesz coś pod **On Fall Triggered**).

---

## Type 2 — Timed Trapdoor

**Co się dzieje:** Gracz wchodzi na normalnie wyglądającą podłogę. Po chwili klapa otwiera się pod jego stopami i wpada.

Klasyczna pułapka ukryta — gracz nie wie, że podłoga jest pułapką, dopóki nie jest za późno.

**Setup:**

Mesh: zamknięta klapa (wygląda jak normalny floor tile). Animator: stany `Idle_Closed` → `Opening` → `Idle_Open`.

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.8–1.5 s *(daj graczowi chwilę zanim klapa pójdzie)* |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |
| **Trap Animator** | Animator z `TrapMesh` |
| **Open Trigger** | `Open` |

!!! tip
    Im dłuższy **Trigger Delay**, tym bardziej okrutna pułapka — gracz ma czas zacząć panikować, ale nie zdąży wyjść.

---

## Type 3 — Lever-Controlled Trapdoor

**Co się dzieje:** Gracz stoi na klapie. Ktoś (albo gracz sam) przekłada dźwignię w innym miejscu — klapa otwiera się i gracz wpada.

Klasyczna pułapka na zaufanie — zaufaj dźwigni i wpadnij.

**Setup:**

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Trigger Delay** | 0.2 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ✅ |
| **Trap Animator** | Animator z `TrapMesh` |

Na dźwigni (`WallLever` lub `WallChain`): przeciągnij `TrapDoorTrap` do listy **Targets**.

!!! tip
    Umieść dźwignię w tym samym pomieszczeniu co klapa — gracz musi zdecydować czy stać na klapie i ciągnąć, czy najpierw odejść i sprawdzić co dźwignia robi.

---

## Type 4 — Lever Puzzle (Cycling Hatches)

**Co się dzieje:** Na drodze stoi kilka klap. Jedna dźwignia przełącza ich stan — za każdym przełożeniem inna kombinacja jest otwarta/zamknięta. Gracz musi znaleźć kombinację w której wszystkie klapy na ścieżce są zamknięte i można bezpiecznie przejść.

**Setup:**

Użyj jednej `WallLever` (toggle). Podepnij do niej kilka `TrapDoorTrap` jako **Targets**.

Każda klapa ma inne ustawienia **Trigger Delay** — to sprawi że nie otwierają się wszystkie jednocześnie, tylko sekwencyjnie, co daje efekt "przesuwania się" kombinacji:

| Klapa | Auto Trigger | Require Party On Cell | Trigger Delay | Permanent | Trigger Once |
|---|---|---|---|---|---|
| Klapa A | ❌ | ❌ | 0 s | ❌ | ❌ |
| Klapa B | ❌ | ❌ | 0.3 s | ❌ | ❌ |
| Klapa C | ❌ | ❌ | 0.6 s | ❌ | ❌ |

Dźwignia aktywuje → klapy otwierają się jedna po drugiej. Dźwignia deaktywuje → zamykają się. Gracz przekłada dźwignię wielokrotnie i obserwuje która kombinacja pozwala przejść.

!!! note
    Ustaw **Permanent = false** i **Trigger Once = false** na każdej klapie — inaczej po pierwszym wyzwoleniu nie będzie można ich resetować.

---

## Type 5 — Logic Gate Puzzle (Multi-Lever Lock)

**Co się dzieje:** Trzy dźwignie kontrolują jedną lub więcej klap. Klapa otwiera się tylko gdy wszystkie trzy dźwignie są w złej pozycji — gracz musi znaleźć jedyną bezpieczną kombinację (wszystkie w górę, wszystkie w dół, lub mieszaną).

**Setup:**

Dodaj komponent `TriggerLogicGate` (typ **AND**) do osobnego GameObject. Podepnij wszystkie trzy `WallLever` jako inputs. Wyjście `TriggerLogicGate` podepnij do `TrapDoorTrap`.

Klapa otwiera się gdy wszystkie dźwignie są aktywne jednocześnie. Bezpieczna pozycja = nie wszystkie aktywne.

Możesz odwrócić logikę używając bramki **NOT** lub **NOR** — klapa jest domyślnie otwarta i zamyka się dopiero gdy gracz ustawi właściwą kombinację.

| Pole na TrapDoorTrap | Wartość |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Permanent** | ❌ |
| **Trigger Once** | ❌ |

---

## Type 6 — Pressure Plate Trap

**Co się dzieje:** Gracz stoi na płytce ciśnieniowej — klapa w innym miejscu jest otwarta. Kiedy gracz schodzi z płytki (albo ktoś inny na niej stanie), klapa zamyka się lub otwiera. Klasyczna "trzymaj płytkę żeby przejść" — ale z akcentem: płytka otwiera klapę pod graczem numer dwa.

**Setup:**

`PressurePlate` → **Targets** → `TrapDoorTrap`.

| Pole na PressurePlate | Wartość |
|---|---|
| *(brak dodatkowych pól — płytka aktywuje się gdy ktoś na niej stoi)* | |

| Pole na TrapDoorTrap | Wartość |
|---|---|
| **Auto Trigger On Enter** | ❌ |
| **Require Party On Cell On External Trigger** | ✅ |
| **Trigger Delay** | 0 s |
| **Permanent** | ✅ |
| **Trigger Once** | ✅ |

Gracz numer jeden staje na płytce → klapa otwiera się. Gracz "numer dwa" (ta sama drużyna, inny moment) wchodzi na klap → wpada.

---

## Type 7 — Repeating Pit

**Co się dzieje:** Klapa otwiera się pod stopami gracza, spada, ginie. Po kilku sekundach klapa zamyka się i pułapka resetuje — czeka na następnego.

Przydatne w korytarzach które gracz musi przechodzić wielokrotnie (powroty, respawn punkty).

**Setup:**

| Pole | Wartość |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **Damage Preset** | InstantKill |
| **Permanent** | ❌ |
| **Close Delay** | 3 s |
| **Trigger Once** | ❌ |
| **Trap Animator** | Animator z `TrapMesh` |

---

## Type 8 — Crossbow Corridor

**Co się dzieje:** Gracz wchodzi w korytarz. Wyzwala pressure plate. Z boku wylatuje strzała.

Klasyczna pułapka ze strzałami — żadnej klapy, tylko `ProjectileTrap` + `PressurePlate`.

**Setup:**

`PressurePlate` → **Targets** → `ProjectileTrap`.

| Pole na ProjectileTrap | Wartość |
|---|---|
| **Projectile Prefab** | prefab strzały / bełtu |
| **Fire Direction** | kierunek prostopadły do ruchu gracza |
| **Damage** | np. 20–30 |
| **Aim Target** | Transform z `PartyVisuals` |

Ustaw kilka `ProjectileTrap` po obu stronach korytarza z różnymi **Fire Direction** — strzały lecą z lewej i prawej jednocześnie.

---

## Type 9 — Pit + Crossbow Combo

**Co się dzieje:** Gracz wchodzi w komórkę z klapą. Klapa otwiera się pod nim. W tej samej chwili z boku wylatuje strzała. Nawet jeśli unika klapy (lever-controlled, nie auto), dostaje strzałą.

Kombinacja `TrapDoorTrap` + `ProjectileTrap` wyzwalanych z tego samego źródła.

**Setup:**

Jedna `WallLever` → **Targets**: `TrapDoorTrap` + `ProjectileTrap` jednocześnie.

Albo: `TrapDoorTrap` ma wypełniony **On Fall Triggered** → wywołuje `ProjectileTrap.Trigger()` przez UnityEvent. Strzała leci dopiero gdy klapa się otworzy.

| Pole na TrapDoorTrap | Wartość |
|---|---|
| **Auto Trigger On Enter** | ✅ |
| **Trigger Delay** | 0.3 s |
| **On Fall Triggered** | → `ProjectileTrap.Trigger()` |

| Pole na ProjectileTrap | Wartość |
|---|---|
| **Fire Direction** | z boku |
| **Damage** | 25 |

---

## Type 10 — Gauntlet

**Co się dzieje:** Gracz musi przejść przez korytarz wypełniony klapami i strzałami. Klapy otwierają się kolejno (każda z innym Trigger Delay), strzały lecą w rytmie z pressure plate na początku korytarza. Bezpieczna ścieżka istnieje, ale trzeba ją znaleźć.

Łączy Type 4 (cycling hatches), Type 7 (repeating) i Type 8 (crossbow corridor).

**Setup:**

1. Rozłóż 4–6 komórek `TrapDoor` wzdłuż korytarza — każda z innym **Trigger Delay** (0 s, 0.5 s, 1 s, 1.5 s…) i **Permanent = false**, **Trigger Once = false**.
2. Jedną `WallLever` podepnij do wszystkich klap — przekładanie dźwigni zmienia która klapa jest otwarta w danym momencie.
3. Na wejściu do korytarza postaw `PressurePlate` → `ProjectileTrap` po bokach.
4. Gracz najpierw sprawdza rytm klap, potem wchodzi i biegnie przez okno bezpieczeństwa.

!!! tip
    Dodaj **On Fall Triggered** → `TriggerPlaySound` na każdej klapie — odgłos spadania gracza słyszany z drugiego końca korytarza buduje napięcie.

---

*CrawlerKIT — Mantis3de*
