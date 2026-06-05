# Setup Guide — Pit & Trapdoor

Każda dziura w podłodze — zwykły otwarty dół albo animowana klapa — używa tego samego typu celi **TrapDoor**. Różnica jest tylko w tym, czy dodajesz komponent `TrapDoorTrap`:

| Wariant | Typ celi | Komponent | Efekt |
|---|---|---|---|
| Zwykły dół | TrapDoor | brak | Drużyna wchodzi i natychmiast spada |
| Pułapka z klapą | TrapDoor | `TrapDoorTrap` | Pełna mechanika: opóźnienie, animacja, lever, reset, obrażenia |

---

## Krok 1 — Oznacz celę w Dungeon Generatorze

W panelu **Spawn Tools** kliknij pomarańczowy przycisk **Trap Door**. To stawia celę z `cellType = TrapDoor` w origin — przesuń ją na właściwe miejsce. Możesz też wybrać istniejącą celę i zmienić jej **Cell Type** na `TrapDoor` w Inspectorze.

Po postawieniu wszystkich celi kliknij **Generate Unique IDs (all traps)**, potem **Build Grid (Export JSON)**.

!!! note "Nie ma już typu Pit"
    Stary typ **Pit** (nieprzejezdny, dekoracyjny) jest wycofany. Wszystkie dziury w podłodze używają **TrapDoor**. Zwykły dół bez komponentu `TrapDoorTrap` zachowuje się dokładnie tak samo jak stary Pit — możesz dodać mechanikę pułapki w każdej chwili, po prostu dokładając komponent.

---

## Krok 2 — Hierarchia prefabu

### Zwykły dół (bez komponentu)

Postaw mesh dołu na celi. Żadnych komponentów nie trzeba. Drużyna wchodzi i spada. Koniec.

### Klapa z mechaniką (z komponentem)

```
TrapDoor              ← root — tutaj idą TrapMarker i TrapDoorTrap
└── TrapMesh          ← geometry klapy + Animator (opcjonalnie)
```

Na **roota** dodaj dwa komponenty: `TrapMarker` i `TrapDoorTrap`.

**`TrapMarker`** — wymagany. Przechowuje `trapId` i rejestruje pułapkę w eksporcie JSON (Save System). Bez niego stan klapy nie będzie zapisywany.

**`TrapDoorTrap`** — właściwa logika pułapki (opóźnienie, obrażenia, animacja, eventy).

Na **TrapMesh** (dziecko) — `Animator` z klipami otwarcia i zamknięcia. Jeśli nie chcesz animacji, zostaw bez Animatora.

---

## Krok 3 — Konfiguracja TrapDoorTrap

### Sekcja: Auto-Trigger on Cell Enter

| Pole | Domyślnie | Co robi |
|---|---|---|
| **Auto Trigger On Enter** | ✅ | Klapa odpala się sama, gdy drużyna wejdzie na celę. Wyłącz jeśli klapa ma być sterowana leverem. |
| **Require Party On Cell On External Trigger** | ✅ | Tylko dla trybu levera. Lever działa **tylko gdy drużyna stoi na klapie**. Wyłącz, jeśli lever ma otwierać klapę niezależnie od pozycji drużyny. |
| **Trigger Delay** | 0 s | Opóźnienie między wejściem na celę a otwarciem klapy (w sekundach). `0` = natychmiast. `0.3–0.5 s` jest wygodne przy animacji otwierania. |

### Sekcja: Behaviour

| Pole | Domyślnie | Co robi |
|---|---|---|
| **Permanent** | ✅ | Klapa zostaje otwarta po odpaleniu. Wyłącz jeśli ma się zamykać po chwili i resetować. |
| **Close Delay** | 2 s | Ile sekund klapa czeka otwarta zanim się zamknie. Działa tylko gdy **Permanent = false**. |
| **Trigger Once** | ✅ | Po pierwszym odpaleniu ignoruje kolejne wejścia i triggery. Wyłącz dla pułapki, która resetuje się i poluje znowu. |

### Sekcja: Fall Damage

| Pole | Domyślnie | Co robi |
|---|---|---|
| **Damage Preset** | InstantKill | `InstantKill` — wszyscy giną natychmiast. `Heavy` — 50 dmg. `Light` — 20 dmg. `Custom` — wartość z pola poniżej. |
| **Fall Damage** | 20 | Wartość obrażeń przy presecie `Custom`. |

### Sekcja: Animator

Wypełnij tylko jeśli masz animowaną klapę. Przy zwykłym otwartym dole zostaw puste.

| Pole | Co wpisać |
|---|---|
| **Trap Animator** | Przeciągnij komponent `Animator` z `TrapMesh` |
| **Open Trigger** | Nazwa triggera w Animatorze który otwiera klapę. Domyślnie: `Open` |
| **Close Trigger** | Nazwa triggera który zamyka klapę. Domyślnie: `Close`. Używany tylko gdy **Permanent = false** lub lever zamyka klapę. |

### Sekcja: Events

| Event | Kiedy się odpala |
|---|---|
| **On Fall Triggered** | Tuż przed aplikacją obrażeń — drużyna właśnie wpada w dziurę. Podepnij tu dźwięk, trzęsienie kamery, cutscenę. |
| **On Trap Closed** | Gdy klapa się zamknie (tylko przy **Permanent = false** lub lever-close). |

---

## Krok 4 — Animator (szczegółowo)

Jeśli klapa ma się wizualnie otwierać i zamykać, potrzebujesz Animatora na `TrapMesh`.

### Stany (States)

Stwórz następujące stany w Animator Controller:

```
Idle_Closed     ← stan startowy (default state) — klapa wygląda jak normalny kafel podłogi
Opening         ← animacja otwierania klapy
Idle_Open       ← klapa otwarta, widać dziurę
Closing         ← animacja zamykania (tylko jeśli Permanent = false lub lever zamyka)
```

Ustaw **Idle_Closed** jako domyślny stan (prawy klik → Set as Layer Default State).

### Parametry (Parameters)

W zakładce **Parameters** Animatora dodaj dwa triggery:

| Nazwa | Typ |
|---|---|
| `Open` | Trigger |
| `Close` | Trigger |

(Jeśli zmieniłeś nazwy w Inspectorze `TrapDoorTrap`, użyj dokładnie tych samych nazw.)

### Przejścia (Transitions)

**Idle_Closed → Opening**

- Warunek: trigger `Open`
- **Has Exit Time**: ❌ wyłączone
- **Transition Duration**: 0

**Opening → Idle_Open**

- **Has Exit Time**: ✅ włączone (przejście po zakończeniu animacji)
- Brak warunku — przechodzi automatycznie po skończeniu klipu

**Idle_Open → Closing** *(tylko jeśli klapa ma się zamykać)*

- Warunek: trigger `Close`
- **Has Exit Time**: ❌ wyłączone
- **Transition Duration**: 0

**Closing → Idle_Closed** *(tylko jeśli klapa ma się zamykać)*

- **Has Exit Time**: ✅ włączone
- Brak warunku — przechodzi automatycznie po skończeniu klipu

!!! tip "Skrócona wersja (bez zamykania)"
    Jeśli klapa jest permanentna (nigdy się nie zamyka), potrzebujesz tylko stanów `Idle_Closed`, `Opening`, `Idle_Open` i triggera `Open`. Stany `Closing` i trigger `Close` możesz pominąć.

---

## Wariant A — Pułapka automatyczna (gracz wchodzi i spada)

To klasyczny ukryty dół: gracz wchodzi, po chwili klapa odpada pod nogami.

**Ustawienia TrapDoorTrap:**

| Pole | Wartość |
|---|---|
| Auto Trigger On Enter | ✅ |
| Trigger Delay | 0.3–0.8 s |
| Damage Preset | InstantKill |
| Permanent | ✅ |
| Trigger Once | ✅ |
| Trap Animator | Animator z TrapMesh |
| Open Trigger | `Open` |

Levera nie potrzebujesz. Koniec setupu.

---

## Wariant B — Lever steruje klapą (toggle: otwiera i zamyka)

Lever otwiera klapę przy pierwszym pociągnięciu. Drugie pociągnięcie zamyka ją z powrotem.

### Na prefabie klapy (TrapDoorTrap):

| Pole | Wartość |
|---|---|
| Auto Trigger On Enter | ❌ |
| Require Party On Cell On External Trigger | ✅ *(drużyna musi stać na klapie)* lub ❌ *(lever działa z dowolnej pozycji)* |
| Trigger Delay | 0.2 s |
| Damage Preset | InstantKill |
| Permanent | ✅ *(klapa zostaje otwarta po odpaleniu — zamknięcie przez lever)* |
| Trap Animator | Animator z TrapMesh |
| Open Trigger | `Open` |
| Close Trigger | `Close` |

### Na leverze (WallLever):

1. Zaznacz GameObject z `WallLever`.
2. W Inspectorze znajdź pole **Targets** (lista).
3. Kliknij **+** i przeciągnij GameObject z komponentem `TrapDoorTrap` do slotu.

Gotowe. Pierwsze pociągnięcie levera → `OnTriggered` → klapa się otwiera (`Open` trigger). Drugie pociągnięcie levera (deaktywacja) → `OnReleased` → klapa się zamyka (`Close` trigger).

!!! warning "Animator musi mieć stan Closing"
    Przy trybie levera z zamykaniem Animator potrzebuje stanów `Idle_Open → Closing → Idle_Closed` i triggera `Close`. Bez tego zamknięcie zadziała w kodzie, ale nie zobaczysz animacji.

---

## Podsumowanie — co gdzie idzie

| GameObject | Komponenty |
|---|---|
| `TrapDoor` (root) | `TrapMarker` *(wymagany)*, `TrapDoorTrap` *(opcjonalny)* |
| `TrapMesh` (dziecko) | `Animator` *(opcjonalny — tylko animowana klapa)* |

Cela na tej samej pozycji w gridzie musi mieć `cellType = TrapDoor` i być wyeksportowana do JSON.

---

*CrawlerKIT — Mantis3de*
