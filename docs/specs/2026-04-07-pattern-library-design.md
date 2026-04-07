# Pattern Library dla Pencil MCP — dokument projektowy

**Data:** 2026-04-07
**Status:** Draft (do akceptacji użytkownika)
**Kontekst projektu:** HouseKeep (React Native + Expo + Gluestack) i przyszłe projekty korzystające z Pencil MCP

## 1. Problem

W aktualnym workflow designu + kodowania z Pencil MCP każdy nowy ekran jest projektowany i kodowany od zera, nawet jeśli zawiera bloki (nawigacja, listy z filtrami, footery, puste stany), które już wielokrotnie powstały w innych projektach lub wcześniejszych iteracjach tego samego projektu. Powoduje to:

- Powtarzalną pracę projektową nad tymi samymi układami
- Niespójność wizualną między ekranami i projektami
- Utratę sprawdzonych rozwiązań implementacyjnych (hooki, state, obsługa błędów)
- Długie sesje projektowe nad rzeczami, które już istnieją

## 2. Cel

Stworzyć **bibliotekę wzorców projektowo-kodowych** (`pattern library`), która pozwala:

- Szybko znaleźć i rozpoznać istniejący wzorzec pasujący do bieżącego zadania
- Jednocześnie ponownie użyć **designu** (klonując Frame z `.pen`) i **kodu** (kopiując `.tsx` z adaptacją)
- Tworzyć nowe rzeczy tylko wtedy, gdy żaden wzorzec nie pasuje
- Rosnąć naturalnie, z minimalnym tarciem przy zapisie nowych wzorców

## 3. Zasady projektowe (non-goals)

**Co to jest:**
- Biblioteka wzorców powyżej poziomu atomów (sekcje i kompozycje)
- Mechanizm wyszukiwania sterowany tagami i wizualnymi podglądami
- Skill orkiestrujący zapis i użycie wzorców

**Czego nie robimy (YAGNI):**
- **Nie duplikujemy atomów Gluestack UI** (`Button`, `Input`, `Box`, itd.) — są już "rozwiązane" przez istniejący design system
- **Nie budujemy własnego MCP serwera** — wzorce to zwykłe pliki na dysku, Pencil MCP jest używany przez swoje istniejące narzędzia (`batch_get`, `batch_design`, `export_nodes`, `open_document`)
- **Nie tworzymy kompozycji w v1** — folder istnieje, ale jest pusty; kompozycje dochodzą dopiero po zaobserwowaniu realnej duplikacji
- **Nie abstrahujemy kodu do "szablonów z placeholderami"** — `implementation.tsx` jest kopiowany surowy, z oryginalnymi importami i kontekstem
- **Nie budujemy indeksu wizualnego po embeddingsach** — wyszukiwanie jest deterministyczne po tagach z kontrolowanej listy

## 4. Architektura

### 4.1 Lokalizacja

Biblioteka wzorców żyje **poza pojedynczym projektem**, aby była dostępna z każdego kodu, który korzysta z Pencila:

```
C:/Users/getrk/patterns/          ← dane biblioteki (wzorce)
C:/Users/getrk/Projects/pattern-library/  ← meta-projekt (spec, plan, kod skill'a)
```

Skill `save-pattern` oraz logika wyszukiwania odwołują się do `C:/Users/getrk/patterns/` przez ścieżkę absolutną.

### 4.2 Struktura katalogów biblioteki

```
patterns/
├── README.md                 ← opis biblioteki, reguła "kiedy tworzyć kompozycję"
├── INDEX.md                  ← płaska tabela: nazwa | kategoria | tagi | ścieżka | opis
├── TAGS.md                   ← kontrolowana lista tagów (~30-50 wpisów)
│
├── sections/                 ← reużywalne bloki (organizmy w nomenklaturze Atomic Design)
│   ├── navigation/
│   │   ├── top-bar-simple/
│   │   │   ├── design.json
│   │   │   ├── preview.png
│   │   │   ├── implementation.tsx
│   │   │   └── README.md
│   │   ├── bottom-tab-5/
│   │   └── drawer-sidebar/
│   ├── hero/
│   ├── content/              ← listy, grid, feature-showcase
│   ├── forms/
│   ├── footer/
│   └── empty-states/
│
└── compositions/             ← PUSTY w v1; przepisy składające sekcje (bez duplikacji danych)
```

### 4.3 Anatomia jednego wzorca (sekcji)

Każda sekcja to katalog z **dokładnie czterema plikami** o stałych nazwach:

| Plik | Zawartość | Generowany przez |
|---|---|---|
| `design.json` | JSON-serializowane drzewo węzłów wzorca (wynik `batch_get` z pełną głębokością) | `batch_get(filePath=source, nodeIds=[source_id], readDepth=999, resolveInstances=true)` + `Write` jako JSON |
| `preview.png` | Eksport wizualny wzorca, standardowy scale 2x | `mcp__pencil__export_nodes` + `mv` z `{node_id}.png` do `preview.png` |
| `implementation.tsx` | Surowy kod React Native z oryginalnymi importami + nagłówek-komentarz | `Read` źródła + `Write` z doklejonym nagłówkiem |
| `README.md` | Frontmatter (metadane, w tym `source_node_id`) + opis "kiedy używać / kiedy nie" + punkty adaptacji | Claude generuje draft, użytkownik akceptuje |

**Kluczowa decyzja architektoniczna (odkryta przy pre-discovery Pencil MCP):** Wzorce nie są zapisywane jako standalone pliki `.pen`. Pencil MCP nie wspiera tworzenia nowego `.pen` pod dowolną ścieżką (`batch_design` operuje na aktywnym dokumencie, `open_document` tylko otwiera istniejące pliki). Zamiast tego wzorzec jest **JSON-serializowanym snapshot'em drzewa węzłów**, który można zrekonstruować w dowolnym dokumencie docelowym przez sekwencję operacji `batch_design I(...)`. `design.json` jest self-contained i portable.

**Trzy warianty wzorców (v1.2):**

1. **Variant 1 — Pencil-sourced (standardowy, 4 pliki):** `design.json` z drzewem węzłów, `preview.png` z `export_nodes`, jeden `implementation.<ext>`, `README.md`. Tak jak opisane powyżej.

2. **Variant 2 — Code-only (3 pliki):** Wzorce z istniejącego kodu bez oddzielnego Pencil designu (np. z istniejących projektów w Laragon). `design.json` zawiera sentinel object `{"_source_type":"code-only","_preview_status":"pending","nodes":[]}` zamiast drzewa węzłów. Brak `preview.png` — backfill później przez dev-browser MCP screenshot lub jeśli powstanie Pencil design. README ma pole `preview_status: pending` w frontmatter.

3. **Variant 3 — Multi-file (4+ plików):** Wzorce interaktywne (FAQ, slidery, mapy, menu mobilne, dropdowny) które wymagają dodatkowych plików JS/CSS obok markupu. Może być Pencil-sourced lub code-only. Layout: **flat** (`implementation.blade.php` + `implementation.js` w jednym folderze) jeśli ≤2 pliki kodu, **nested** (`implementation/` subdir z `view.<ext>` + `script.js` + `style.scss` + ewentualne dep-*) jeśli ≥3 pliki kodu. Skill w Kroku 5b automatycznie przeszukuje `resources/js/` i `resources/css/` projektu źródłowego pod kątem powiązanych plików i pyta użytkownika co dołączyć.

Wszystkie trzy warianty współistnieją w tej samej bibliotece — są rozróżnialne przez frontmatter `source_type` w README.md i obecność/brak `preview.png`. Bramka 2 wyszukiwania (wizualny podgląd) jest no-op dla Variant 2/3 bez preview — wtedy Claude od razu przechodzi do Bramki 3 (przegląd kodu).

Brak któregokolwiek z tych plików = wzorzec jest niekompletny i nie pojawia się w wyszukiwaniu. Atomowość zapisu (patrz 5.2) gwarantuje, że nigdy nie powstaje pół-wzorzec.

### 4.4 Trzybramkowy pipeline wyszukiwania wzorca

Wyszukiwanie jest **kaskadowe od taniego do drogiego** — każda bramka jest stop/go:

```
Zadanie: "zrób ekran ustawień z grupami opcji"
         │
         ▼
┌─────────────────────────────────────────────┐
│ BRAMKA 1 — TAGI / INDEX                     │
│ Grep patterns/INDEX.md po tagach z zadania  │
│ Koszt: ~0 (mały plik tekstowy)              │
└─────────────────────────────────────────────┘
         │
     miss ├──► STOP: brak wzorca, twórz od zera
         │
     hit  ▼
┌─────────────────────────────────────────────┐
│ BRAMKA 2 — PODGLĄD WIZUALNY (PNG)           │
│ Read preview.png każdego kandydata          │
│ Claude ocenia: "czy layout pasuje?"         │
│ Koszt: 1 tool-call na kandydata              │
└─────────────────────────────────────────────┘
         │
  no match├──► następny kandydat z listy wygenerowanej przez Bramkę 1
         │
  match   ▼
┌─────────────────────────────────────────────┐
│ BRAMKA 3 — PRZEGLĄD KODU                    │
│ Read implementation.tsx (z nagłówkiem)      │
│ Claude ocenia: "czy zależności i struktura  │
│ pasują do bieżącego projektu?"              │
│ Koszt: 1 Read na kandydata                  │
└─────────────────────────────────────────────┘
         │
  no fit  ├──► STOP: użyj tylko designu jako inspiracji,
         │                   kod pisz od nowa
  fit     ▼
┌─────────────────────────────────────────────┐
│ GENEROWANIE                                 │
│ 1. Read design.json (drzewo węzłów wzorca)  │
│ 2. Upewnij się, że docelowy .pen jest       │
│    aktywny w edytorze (get_editor_state)    │
│ 3. Zdeserializuj JSON → sekwencja operacji  │
│    batch_design I(...) w kolejności od      │
│    rodzica do dzieci (≤25 ops na call)      │
│ 4. Wykonaj batch_design na aktywnym .pen    │
│ 5. Adaptacja implementation.tsx do bieżącego│
│    projektu (typy, importy, punkty z header)│
│ Koszt: największy — właściwe użycie wzorca  │
└─────────────────────────────────────────────┘
```

**Kluczowa własność:** najdroższa operacja (odczyt `.pen` i generowanie) wykonuje się tylko wtedy, gdy poprzednie trzy tańsze bramki wskazały, że warto.

### 4.5 Tagowanie — kontrolowana lista

Tagi są deterministyczne. Wolne tagi by prędko spowodowały duplikaty (`nav` vs `navigation` vs `navbar`), co zepsułoby bramkę 1.

`patterns/TAGS.md` zawiera pełną listę dozwolonych tagów pogrupowanych semantycznie:

```markdown
# Kontrolowana lista tagów

## Nawigacja
- `navigation` — dowolna forma nawigacji główna
- `tabs` — tab bar (dolny lub górny)
- `drawer` — menu wysuwane z boku
- `header` — górny pasek (z tytułem, akcjami)
- `breadcrumbs`

## Struktura treści
- `list` — lista pionowa
- `grid` — siatka
- `card-list` — lista kart
- `feature-showcase`
- `hero`
- `empty-state`

## Interakcja
- `form` — formularz
- `search` — pole wyszukiwania
- `filters` — chipsy/dropdowny filtrów
- `toggles` — przełączniki (dla ekranów ustawień)
- `fab` — floating action button

## Domena
- `settings`
- `auth` — logowanie/rejestracja
- `onboarding`
- `profile`
- `notifications`

## Stan
- `loading`
- `error`
- `success`
```

**Reguła nadawania tagów:** Claude przy zapisie wzorca czyta `TAGS.md`, wybiera pasujące tagi **wyłącznie z listy**. Jeśli żaden istniejący tag nie pasuje, Claude proponuje użytkownikowi nowy wpis do listy i czeka na akceptację (nie dodaje sam).

## 5. Skill `save-pattern`

### 5.1 Interfejs użytkownika

Skill jest wywoływany przez użytkownika w języku naturalnym, np.:

> "Zapisz ten ekran jako wzorzec 'task-list-with-filters'"

Claude:
1. Wykrywa z `get_editor_state` aktualnie otwarty `.pen` i zaznaczony węzeł
2. Pyta o kategorię (`sections/navigation`, `sections/content`, itp.) — z sugestią na podstawie struktury
3. Pyta, który plik `.tsx` jest implementacją tego wzorca (jeśli nie jest pewien z kontekstu konwersacji)
4. Pyta o opcjonalny krótki opis
5. Generuje draft README i pokazuje go do akceptacji
6. Po akceptacji wykonuje kroki zapisu (5.2) atomowo

### 5.2 Sekwencja zapisu (7 kroków, atomowa)

```
Krok 1 — Zbierz źródła
  • get_editor_state → aktualny .pen + ID zaznaczonego węzła
  • Od użytkownika: ścieżka do .tsx, nazwa, kategoria, tagi (z TAGS.md)
  • Czytanie TAGS.md i walidacja wybranych tagów

Krok 2 — Utwórz strukturę folderu
  • mkdir patterns/<kategoria>/<nazwa>/
  • Weryfikacja, że nazwa nie koliduje z istniejącym wzorcem

Krok 3 — Zapisz design.json (JSON-serializowane drzewo węzłów)
  • W aktywnym edytorze jest źródłowy .pen z zaznaczonym węzłem
  • batch_get(filePath=<SOURCE_PEN>, nodeIds=[<SOURCE_NODE_ID>],
             readDepth=999, resolveInstances=true)
    → pełne drzewo wzorca jako obiekt JSON
  • Write patterns/<kategoria>/<nazwa>/design.json z wynikiem batch_get

Krok 4 — Wyeksportuj preview.png
  • export_nodes(filePath=<SOURCE_PEN>, outputDir=<katalog wzorca>,
                 nodeIds=[<SOURCE_NODE_ID>], format="png", scale=2)
    → tworzy <SOURCE_NODE_ID>.png w katalogu wzorca
  • mv <SOURCE_NODE_ID>.png → preview.png (export_nodes nie pozwala
    wybrać nazwy pliku, więc rename jest obowiązkowy)

Krok 5 — Skopiuj implementation.tsx
  • Read oryginalnego pliku .tsx
  • Wygeneruj nagłówek-komentarz (patrz 6.3)
  • Write patterns/<kategoria>/<nazwa>/implementation.tsx z nagłówkiem + oryginalnym kodem

Krok 6 — Wygeneruj README.md
  • Claude tworzy draft z frontmatter + sekcje ("Co to jest", "Kiedy używać",
    "Kiedy NIE używać", "Zależności", "Punkty adaptacji")
  • Draft pokazany użytkownikowi do akceptacji/edycji
  • Po akceptacji: Write do patterns/<kategoria>/<nazwa>/README.md

Krok 7 — Zaktualizuj INDEX.md
  • Read INDEX.md
  • Dodaj nowy wiersz: | nazwa | kategoria | tagi | ścieżka | opis |
  • Write zaktualizowany INDEX.md
  • Jeśli ten krok zawiedzie (np. błąd zapisu), skill również usuwa
    świeżo utworzony folder wzorca — patrz reguła atomowości poniżej
```

**Atomowość:** Jeśli którykolwiek krok zawiedzie (np. `export_nodes` zwraca błąd, użytkownik odrzuca README, błąd zapisu INDEX), skill usuwa cały katalog `patterns/<kategoria>/<nazwa>/` i nie modyfikuje INDEX.md. Stan biblioteki zawsze jest spójny — nigdy nie pozostaje po połowie zapisany wzorzec ani INDEX z wpisem wskazującym na niekompletny folder.

### 5.3 Interakcja z użytkownikiem

Skill rozgranicza operacje "deterministyczne" (nie wymagają akceptacji) od "kreatywnych" (wymagają):

| Operacja | Rodzaj | Akceptacja? |
|---|---|---|
| Kopia `.pen` node | Deterministyczna | Nie |
| Eksport PNG | Deterministyczna | Nie |
| Kopia `.tsx` | Deterministyczna | Nie |
| Nagłówek metadanych | Deterministyczny (czytany z kodu) | Nie |
| Tagi | Wybór z kontrolowanej listy | Tak (tylko jeśli proponowane nowe tagi) |
| README — frontmatter | Deterministyczny | Nie |
| README — opis i "Kiedy używać" | Kreatywny | **Tak — draft + akceptacja** |
| INDEX wiersz | Deterministyczny | Nie |

## 6. Formaty plików

### 6.1 INDEX.md

Płaska tabela Markdown, jeden wiersz na wzorzec. Cel: tani, szybki grep tagów.

```markdown
# Pattern Library Index

| Nazwa | Kategoria | Tagi | Ścieżka | Opis |
|---|---|---|---|---|
| bottom-tab-5 | navigation | navigation, tabs | sections/navigation/bottom-tab-5 | Dolny pasek nawigacji z 5 zakładkami, ikony MaterialCommunityIcons |
| settings-grouped-list | content | list, settings, toggles | sections/content/settings-grouped-list | Lista grupowana dla ekranu ustawień, z nagłówkami sekcji i przełącznikami |
| task-list-with-filters | content | list, filters, search, fab | sections/content/task-list-with-filters | Lista zadań z wyszukiwarką, chipsami filtrów i przyciskiem dodawania (FAB) |
```

### 6.2 README.md per wzorzec

```markdown
---
name: bottom-tab-5
category: sections/navigation
tags: [navigation, tabs]
created: 2026-04-07
source_project: housekeep
source_file: mobile/src/app/(app)/_layout.tsx
dependencies:
  - expo-router
  - @expo/vector-icons
  - gluestack-ui (Text)
  - hook: useColors (theme)
---

# Bottom Tab Navigation (5 items)

## Co to jest
Dolny pasek nawigacji z 5 zakładkami opartym na `expo-router/Tabs`.
Ikony z `MaterialCommunityIcons`. Kolory zakładek biorą wartości z
hooka `useColors()` (adaptacja do motywu light/dark).

## Kiedy używać
- Aplikacja ma 3-5 głównych sekcji dostępnych z każdego miejsca
- Nie ma potrzeby hierarchii głębszej niż 2 poziomy
- Target: telefon (nie tablet)

## Kiedy NIE używać
- Aplikacja ma >5 głównych sekcji (użyj drawer zamiast tabs)
- Nawigacja jest kontekstowa (zmienia się per ekran)
- Cel: web (użyj top-bar + sidebar)

## Zależności do zainstalowania
- `expo-router`
- `@expo/vector-icons`

## Punkty adaptacji (co zmienić przy kopiowaniu)
- **L8** — etykiety zakładek (obecnie w PL: "Dom", "Produkty", "Zadania", "Zakupy", "Ustawienia")
- **L15** — nazwy ikon z `MaterialCommunityIcons`
- **L22** — nazwy tras, muszą pasować do struktury `expo-router` w twoim projekcie
- **L31** — `useColors()`, podłącz pod swój system motywów (albo zastąp statycznymi kolorami)
```

### 6.3 Nagłówek w `implementation.tsx`

Generowany deterministycznie, nie wymaga akceptacji:

```tsx
// ════════════════════════════════════════════════════════════
// Pattern: sections/navigation/bottom-tab-5
// Source: housekeep (mobile/src/app/(app)/_layout.tsx)
// Saved: 2026-04-07
//
// Dependencies:
//   - expo-router (Tabs)
//   - @expo/vector-icons (MaterialCommunityIcons)
//   - hook: useColors (theme)
//
// Adaptation points (zmień przy kopiowaniu):
//   L8  — etykiety zakładek (obecnie PL: "Dom", "Produkty"...)
//   L15 — nazwy ikon z MaterialCommunityIcons
//   L22 — nazwy tras (musi pasować do struktury expo-router twojego projektu)
//   L31 — useColors() → podłącz pod swój system motywów
// ════════════════════════════════════════════════════════════

// [Tutaj surowy, nietknięty kod z oryginalnego pliku]
import { Tabs } from 'expo-router';
// ...
```

Linie w punktach adaptacji odnoszą się do **linii w kodzie pod nagłówkiem** (nagłówek jest pomijany w numeracji — skill liczy od pierwszej linii kodu).

### 6.4 TAGS.md

Patrz sekcja 4.5.

### 6.5 patterns/README.md (biblioteki)

Krótki opis + kluczowe reguły dla Claude'a w przyszłych sesjach:

```markdown
# Pattern Library

Biblioteka reużywalnych wzorców projektowo-kodowych dla projektów z Pencil MCP.

## Struktura
- `sections/` — reużywalne bloki (nawigacja, listy, formularze, ...)
- `compositions/` — przepisy składające sekcje (pusty w v1)
- `INDEX.md` — tabela wszystkich wzorców (grep po tagach)
- `TAGS.md` — kontrolowana lista tagów

## Reguły dla Claude'a

### Przy projektowaniu nowego ekranu
ZAWSZE najpierw sprawdź bibliotekę w kolejności:
1. Grep INDEX.md po tagach związanych z zadaniem
2. Dla kandydatów: Read preview.png i oceń wizualnie
3. Dla pasujących: Read implementation.tsx i oceń kod
4. Dopiero jeśli nic nie pasuje — twórz od zera

### Przy zapisie wzorca
Użyj skill'a `save-pattern`. Nigdy nie twórz ręcznie plików w patterns/.

### Przy tagowaniu
Wybieraj tagi WYŁĄCZNIE z TAGS.md. Jeśli żaden nie pasuje, zaproponuj użytkownikowi
nowy wpis do listy i poczekaj na akceptację.

### Kiedy tworzyć kompozycję
Tylko jeśli ten sam zestaw 3+ sekcji występuje razem w 2+ projektach.
Przedtem zostaw compositions/ pusty.
```

## 7. Przykładowy pełny cykl użycia

### 7.1 Zapis nowego wzorca

```
[Kontekst: użytkownik właśnie skończył design i kod ekranu listy zadań w HouseKeep]

User: "Ten ekran listy zadań wyszedł dobrze. Zapisz go jako wzorzec."

Claude:
  1. get_editor_state → aktualny .pen to housekeep/mobile/src/screens/tasks.pen,
     zaznaczony węzeł: "TaskListScreen"
  2. "Pod jaką nazwą? Proponuję 'task-list-with-filters'. Kategoria sections/content?"
User: "ok, nazwa i kategoria dobre"
  3. "Który plik .tsx jest implementacją? Widzę w konwersacji mobile/src/screens/TasksScreen.tsx"
User: "ten"
  4. Claude czyta TAGS.md, proponuje tagi: [list, filters, search, fab]
     → wszystkie istnieją w TAGS.md, nie trzeba pytać
  5. Claude generuje draft README i pokazuje
User: "ok, ale w 'Kiedy NIE używać' dopisz że nie pasuje do grup/kategorii"
Claude: [poprawia draft, pokazuje jeszcze raz]
User: "ok"
  6-7. Claude wykonuje 7 kroków zapisu atomowo
Claude: "Zapisano sections/content/task-list-with-filters/. 4 pliki + INDEX zaktualizowany."
```

### 7.2 Użycie wzorca w nowym projekcie

```
[Kontekst: trzy miesiące później, nowy projekt — czytelnia książek]

User: "Zrób ekran listy książek z filtrami po autorze i gatunku"

Claude:
  [Bramka 1] grep patterns/INDEX.md po "list AND filters"
             → trafienia: task-list-with-filters, recipe-list
  [Bramka 2] Read preview.png dla obu
             → task-list-with-filters ma layout najbardziej zbliżony (lista + chipsy)
  [Bramka 3] Read implementation.tsx
             → nagłówek wskazuje: expo-router, gluestack-ui, useColors
             → zależności pasują do nowego projektu
             → punkty adaptacji: L12 zmienić typ Task → Book, L24 zmienić filtry
  [Generowanie]
             → open_document bieżącego .pen
             → Read design.json → transform to batch_design I() operations → insert into active .pen
             → adaptacja kodu: Task → Book, filtry author/genre, reszta bez zmian
Claude: "Użyłem wzorca task-list-with-filters. Zmieniłem 3 rzeczy wskazane w nagłówku.
         Design skopiowany, kod gotowy — sprawdź adaptację typów Book."
```

## 8. Obsługa błędów i przypadków granicznych

| Sytuacja | Zachowanie |
|---|---|
| Brak aktywnego `.pen` w edytorze przy `save-pattern` | Skill pyta użytkownika, który plik otworzyć; jeśli odmówi — abort |
| `export_nodes` zwraca błąd | Rollback: usuń folder `<nazwa>/`, nie modyfikuj INDEX |
| Nazwa wzorca już istnieje | Pytaj użytkownika: nadpisać, podać inną nazwę, anulować |
| Plik `.tsx` nie istnieje lub jest pusty | Abort z jasnym komunikatem — nie zapisuj wzorca bez kodu |
| Użytkownik odrzuca draft README | Loop: Claude proponuje poprawki, maksymalnie 3 iteracje, potem abort |
| Zaproponowany tag spoza `TAGS.md` | Pytaj, czy dodać do listy; bez akceptacji abort |
| `INDEX.md` nie istnieje | Utwórz pusty z nagłówkiem tabeli, potem dodaj wiersz |
| Katalog `patterns/` nie istnieje | Abort z komunikatem "zainicjuj bibliotekę najpierw" |

## 9. Wymagania implementacyjne

### 9.1 Narzędzia Pencil MCP używane przez skill

**Zapis wzorca (save-pattern):**
- `get_editor_state()` — odczyt aktywnego `.pen` (źródłowego) i ID zaznaczonego węzła
- `batch_get(filePath, nodeIds, readDepth, resolveInstances)` — odczyt pełnego drzewa węzła źródłowego jako JSON
- `export_nodes(filePath, outputDir, nodeIds, format, scale)` — eksport wzorca do PNG. **Ważne:** plik wyjściowy ma nazwę `{nodeId}.{ext}` — skill musi przemianować na `preview.png` po eksporcie
- `Write` (nie Pencil MCP) — zapis zserializowanego JSON z `batch_get` do `design.json`

**Użycie wzorca (pipeline wyszukiwania):**
- `Read design.json` — odczyt zserializowanego drzewa węzłów
- `get_editor_state()` — potwierdzenie, że docelowy `.pen` jest aktywnym edytorem (jeśli nie, użytkownik musi go otworzyć ręcznie)
- `batch_design(filePath, operations)` — seria operacji `I(parent, {...node_data...})` rekonstruujących drzewo z JSON w aktywnym dokumencie; potrzeba może być rozbita na wiele wywołań z ≤25 ops każde

**Kluczowe ograniczenia Pencil MCP (odkryte przy pre-discovery):**
- `batch_design` z parametrem `filePath` wskazującym na nieistniejący plik **nie tworzy pliku** — operacje wykonują się na aktywnym edytorze. Dlatego nie używamy `batch_design` do utworzenia standalone `design.pen`.
- `open_document('new')` tworzy pusty dokument w edytorze, ale nie ma mechanizmu zapisania go pod wybraną ścieżką przez MCP.
- `export_nodes` nie pozwala wybrać nazwy pliku wyjściowego — używa ID węzła jako nazwy.

### 9.2 Narzędzia standardowe

- `Read`, `Write`, `Edit` — operacje na `.tsx`, `.md`
- `Grep` — bramka 1 wyszukiwania
- `Bash` (mkdir) — tworzenie struktury katalogu

### 9.3 Lokalizacja plików skill'a

Skill jako pliki Markdown w standardowej lokalizacji skilli Claude Code (do ustalenia w planie implementacji — zależy od konwencji lokalnej instalacji).

## 10. Mierzalne kryteria sukcesu

Po zakończeniu v1 użytkownik powinien móc:

1. Wywołać `save-pattern` na gotowym designie + kodzie w HouseKeep i otrzymać kompletny wzorzec (4 pliki + INDEX) w < 2 minut
2. W nowym projekcie, mając zadanie "zrób X", doprowadzić Claude'a do rozpoznania wzorca z biblioteki i skopiowania go (design + kod) bez ręcznego wskazywania, który wzorzec użyć
3. Mieć rosnącą bibliotekę (>= 5 sekcji po pierwszym tygodniu użycia) bez frustracji związanej z procesem zapisu

## 11. Decyzje do uszczegółowienia w planie implementacji

Rzeczy, które zostały świadomie zostawione planowi implementacji, nie projektowi:

- Dokładna lokalizacja plików skill'a na dysku (zależy od sposobu, w jaki Claude Code ładuje lokalne skille na tej instalacji)
- Sposób uruchamiania skill'a — czy przez komendę slash (`/save-pattern`), czy przez rozpoznawanie fraz ("zapisz jako wzorzec")
- Czy bramka 1 używa `Grep` na `INDEX.md`, czy parsuje Markdown programowo (Grep jest prostszy, ale mniej precyzyjny)
- Pierwsze 3-5 wzorców do zasiania biblioteki w ramach v1 (np. `bottom-tab-5`, `settings-grouped-list`, `task-list-with-filters`)
- Format i dokładna treść pliku startowego `TAGS.md` (pełna lista ~30-50 tagów)

## 12. Zmiany poza zakresem v1 (backlog)

- **Kompozycje** — przepisy składające sekcje, dodajemy gdy zaobserwujemy 3+ sekcji występujących razem w 2+ projektach
- **Wersjonowanie wzorców** — gdy wzorzec ewoluuje, trzymamy historię zamiast nadpisywać
- **Cross-project sync** — jeśli biblioteka ma być współdzielona między maszynami, potrzebny git repo i workflow sync
- **Wizualne wyszukiwanie po embeddingsach** — jeśli tagowanie okaże się niewystarczające
- **Automatyczne wykrywanie duplikatów** — gdy dwa wzorce mają bardzo podobny design lub kod
- **Provenance tags** (zaimplementowane v1.1) — Wzorce z różnych projektów (własne portfolio + projekty klienckie) trafiają do **jednej** biblioteki `C:/Users/getrk/patterns/`, ale każdy ma obowiązkowy provenance tag identyfikujący IP/scope reuse: `gregormedia` (własne portfolio, free reuse), `specialspace` (Cortez, E-balans, LiveFood, …), `bravenew` (HCBio, InfiniteServices, …). Filtrowanie w bramce 1 wyszukiwania po provenance pozwala dopasować reuse do reguł licencyjnych klienta. Provenance per projekt jest weryfikowalne przez `style.css` (pole `Author:`) w głównym folderze theme'u źródłowego. **Poprzednia koncepcja** (osobne biblioteki per klient z forkami skill'a) została odrzucona jako nadmiernie skomplikowana.
