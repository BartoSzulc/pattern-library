# Pattern Library Implementation Plan

> **For agentic workers:** REQUIRED: Use superpowers:subagent-driven-development (if subagents available) or superpowers:executing-plans to implement this plan. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Zbudować bibliotekę wzorców projektowo-kodowych (`pattern library`) dla Pencil MCP, wraz ze skillem `save-pattern`, do momentu w którym pierwszy realny wzorzec z HouseKeep (np. dolny tab bar) jest zapisany end-to-end i dostępny do użycia w przyszłych projektach.

**Architecture:** Biblioteka to zwykłe pliki na dysku pod `C:/Users/getrk/patterns/` (poza projektem, współdzielone). Meta-projekt (spec, plan, skrypty) żyje pod `C:/Users/getrk/Projects/pattern-library/`. Skill `save-pattern` to pojedynczy plik `.md` w `C:/Users/getrk/.claude/skills/save-pattern.md`, który orkiestruje 7-krokowy atomowy zapis wzorca z użyciem narzędzi Pencil MCP.

**Tech Stack:** Markdown (pliki wzorców, dokumentacja), Bash (mkdir, git), Pencil MCP (`get_editor_state`, `open_document`, `batch_design`, `batch_get`, `export_nodes`), Claude Code skill system.

**Referenced spec:** `C:/Users/getrk/Projects/pattern-library/docs/specs/2026-04-07-pattern-library-design.md`

---

## File Structure

**Meta-project (pattern-library):**
```
C:/Users/getrk/Projects/pattern-library/
├── README.md                                          ← krótki opis meta-projektu
├── .gitignore                                         ← ignoruje pliki tymczasowe
├── docs/
│   ├── specs/
│   │   └── 2026-04-07-pattern-library-design.md       ← (już istnieje)
│   └── plans/
│       └── 2026-04-07-pattern-library-implementation.md ← (ten plik)
```

Odpowiedzialność: spec + plan + historia decyzji. Nie zawiera kodu aplikacji, nie zawiera danych biblioteki — tylko dokumentację projektu budowy biblioteki.

**Library (patterns):**
```
C:/Users/getrk/patterns/
├── README.md                 ← reguły dla Claude'a (kiedy szukać, kiedy zapisywać)
├── INDEX.md                  ← tabela wszystkich wzorców
├── TAGS.md                   ← kontrolowana lista tagów
├── sections/
│   ├── navigation/           ← (pusty katalog na start)
│   ├── hero/
│   ├── content/
│   ├── forms/
│   ├── footer/
│   └── empty-states/
└── compositions/             ← (pusty, v1 wyłącznie szkielet)
```

Odpowiedzialność: trzyma wszystkie wzorce i ich metadane. Każdy plik ma jedną rolę. Nie zawiera kodu uruchomieniowego — to jest statyczna baza danych na dysku.

**Skill:**
```
C:/Users/getrk/.claude/skills/
└── save-pattern.md           ← jeden plik, format jak deploy.md (# Skill: save-pattern)
```

Odpowiedzialność: instrukcje dla Claude'a jak przeprowadzić 7-krokowy zapis wzorca, obsługa błędów, reguły tagowania, interakcja z użytkownikiem. Nie zawiera kodu — to prompt template.

---

## Chunk 1: Library Scaffold + Meta-Project Init

Cel chunka: zainicjalizować meta-projekt jako repozytorium git oraz utworzyć kompletną strukturę biblioteki z trzema statycznymi plikami (`README.md`, `INDEX.md`, `TAGS.md`) i pustymi katalogami kategorii.

### Task 1: Initialize meta-project as git repo

**Files:**
- Create: `C:/Users/getrk/Projects/pattern-library/README.md`
- Create: `C:/Users/getrk/Projects/pattern-library/.gitignore`
- Create: `C:/Users/getrk/Projects/pattern-library/.git/` (via `git init`)

- [ ] **Step 1.1: Verify meta-project directory exists**

Run:
```bash
ls -la "C:/Users/getrk/Projects/pattern-library/docs/specs/2026-04-07-pattern-library-design.md"
```
Expected: file exists (potwierdza, że spec jest na miejscu zanim budujemy wokół niego).

- [ ] **Step 1.2: Create meta-project README.md**

Write to `C:/Users/getrk/Projects/pattern-library/README.md`:

```markdown
# Pattern Library (meta-project)

Meta-projekt dla biblioteki wzorców projektowo-kodowych działającej nad Pencil MCP.

## Co tu jest
- `docs/specs/` — dokumentacja projektowa (specyfikacja)
- `docs/plans/` — plany implementacji
- Ten folder NIE zawiera samej biblioteki wzorców. Dane żyją w `C:/Users/getrk/patterns/`.

## Co tu NIE ma
- Kodu aplikacyjnego
- Plików wzorców (`.pen`, `.tsx`, `.png`, `.md`)
- Skill'a `save-pattern` (żyje w `C:/Users/getrk/.claude/skills/save-pattern.md`)

## Zobacz
- Specyfikacja: [docs/specs/2026-04-07-pattern-library-design.md](docs/specs/2026-04-07-pattern-library-design.md)
- Plan implementacji: [docs/plans/2026-04-07-pattern-library-implementation.md](docs/plans/2026-04-07-pattern-library-implementation.md)
```

- [ ] **Step 1.3: Create .gitignore**

Write to `C:/Users/getrk/Projects/pattern-library/.gitignore`:

```
# OS
.DS_Store
Thumbs.db
desktop.ini

# Editor
.vscode/
.idea/
*.swp
*~

# Temp
*.tmp
*.log
```

- [ ] **Step 1.4: Verify all files to commit exist on disk**

Run:
```bash
for f in "C:/Users/getrk/Projects/pattern-library/README.md" \
         "C:/Users/getrk/Projects/pattern-library/.gitignore" \
         "C:/Users/getrk/Projects/pattern-library/docs/specs/2026-04-07-pattern-library-design.md" \
         "C:/Users/getrk/Projects/pattern-library/docs/plans/2026-04-07-pattern-library-implementation.md"; do
  test -f "$f" && echo "OK: $f" || echo "MISSING: $f"
done
```
Expected: cztery linie `OK:`. Jeśli którykolwiek plik jest `MISSING`, przerwij — nie zaczynaj `git init` dopóki wszystkie cztery pliki nie istnieją.

- [ ] **Step 1.5: Initialize git repo**

Run:
```bash
cd "C:/Users/getrk/Projects/pattern-library" && git init && git add README.md .gitignore docs/specs/2026-04-07-pattern-library-design.md docs/plans/2026-04-07-pattern-library-implementation.md && git status
```
Expected: `git status` shows 4 staged files, no errors.

- [ ] **Step 1.6: First commit**

Run:
```bash
cd "C:/Users/getrk/Projects/pattern-library" && git commit -m "chore: initialize pattern-library meta-project with spec and plan"
```
Expected: commit succeeds, 4 files committed.

### Task 2: Create library directory structure

**Files:**
- Create: `C:/Users/getrk/patterns/` (plus subdirectories)

- [ ] **Step 2.1: Verify library parent location does not already exist**

Run:
```bash
test ! -d "C:/Users/getrk/patterns" && echo "OK: does not exist" || echo "FAIL: already exists, inspect manually before continuing"
```
Expected: `OK: does not exist`. Jeśli widzisz `FAIL`, przerwij plan i sprawdź ręcznie zawartość istniejącego katalogu — nie chcemy nadpisać czegoś, co użytkownik tam trzyma. Wznów plan dopiero po rozstrzygnięciu z użytkownikiem.

- [ ] **Step 2.2: Create all directories in one mkdir call**

Run:
```bash
mkdir -p \
  "C:/Users/getrk/patterns/sections/navigation" \
  "C:/Users/getrk/patterns/sections/hero" \
  "C:/Users/getrk/patterns/sections/content" \
  "C:/Users/getrk/patterns/sections/forms" \
  "C:/Users/getrk/patterns/sections/footer" \
  "C:/Users/getrk/patterns/sections/empty-states" \
  "C:/Users/getrk/patterns/compositions"
```
Expected: no output, exit code 0.

- [ ] **Step 2.3: Verify directory tree**

Run:
```bash
find "C:/Users/getrk/patterns" -type d | sort
```
Expected output:
```
C:/Users/getrk/patterns
C:/Users/getrk/patterns/compositions
C:/Users/getrk/patterns/sections
C:/Users/getrk/patterns/sections/content
C:/Users/getrk/patterns/sections/empty-states
C:/Users/getrk/patterns/sections/footer
C:/Users/getrk/patterns/sections/forms
C:/Users/getrk/patterns/sections/hero
C:/Users/getrk/patterns/sections/navigation
```

### Task 3: Write patterns/README.md (library rules for Claude)

**Files:**
- Create: `C:/Users/getrk/patterns/README.md`

- [ ] **Step 3.1: Write the library rules file**

Write to `C:/Users/getrk/patterns/README.md`:

```markdown
# Pattern Library

Biblioteka reużywalnych wzorców projektowo-kodowych (sekcji UI) dla projektów
korzystających z Pencil MCP. Zawiera sparowane pliki designu (`.pen`) i kodu
(`.tsx`) z ustrukturyzowanymi metadanymi, dzięki którym Claude może szybko
rozpoznać i ponownie użyć istniejący wzorzec zamiast tworzyć od zera.

## Struktura
- `sections/` — reużywalne bloki UI pogrupowane semantycznie (navigation,
  hero, content, forms, footer, empty-states). Każdy podkatalog to jedna sekcja
  z dokładnie czterema plikami: `design.json`, `preview.png`, `implementation.tsx`,
  `README.md`.
- `compositions/` — przepisy składające wiele sekcji w gotowe ekrany. W v1
  PUSTY — tworzymy kompozycję dopiero gdy ten sam zestaw 3+ sekcji powtarza
  się w 2+ projektach.
- `INDEX.md` — tabela wszystkich wzorców, używana do szybkiego wyszukiwania
  po tagach (Bramka 1 pipeline'u wyszukiwania).
- `TAGS.md` — kontrolowana lista dozwolonych tagów. Każdy tag użyty w
  jakimkolwiek wzorcu musi znajdować się na tej liście.

## Reguły dla Claude'a

### Zanim zaczniesz projektować cokolwiek nowego
ZAWSZE sprawdź najpierw bibliotekę w trzech krokach (od taniego do drogiego):

1. **Bramka 1 — tagi.** `Grep` po `INDEX.md` dla tagów związanych z zadaniem
   (np. "navigation", "list", "form"). Wynik to lista kandydatów.
2. **Bramka 2 — wizualny podgląd.** Dla każdego kandydata `Read` jego
   `preview.png` i oceń, czy layout pasuje do zadania.
3. **Bramka 3 — przegląd kodu.** Dla kandydatów pasujących wizualnie
   `Read` `implementation.tsx`. Nagłówek-komentarz na górze pliku wylicza
   zależności i punkty adaptacji — użyj ich, żeby ocenić czy kod nadaje się
   do bieżącego projektu.

Dopiero gdy żaden wzorzec nie przejdzie wszystkich trzech bramek, projektuj
od zera. Przy projektowaniu od zera rozważ, czy wynik warto zapisać jako
nowy wzorzec (patrz niżej).

### Przy użyciu wzorca (kopiowanie do bieżącego projektu)
1. `Read <ścieżka design.json wzorca>` — wczytaj zserializowane drzewo węzłów
2. Upewnij się, że docelowy `.pen` projektu jest aktywnym edytorem
   (`get_editor_state` pokaże który dokument jest aktywny; jeśli nie
   ten, użytkownik musi go otworzyć ręcznie w Pencilu)
3. Zdeserializuj JSON i przekształć na sekwencję operacji
   `batch_design I(...)` w kolejności od rodzica do dzieci, maks 25 ops per call
4. Wywołaj `batch_design(filePath=<aktywny_pen>, operations=...)` raz
   lub wiele razy, aż całe drzewo zostanie wstawione
5. Adaptacja `implementation.tsx` w kodzie projektu z uwzględnieniem
   punktów adaptacji z nagłówka-komentarza

### Przy zapisie wzorca
UŻYWAJ skill'a `save-pattern`. Nigdy nie twórz plików w `patterns/` ręcznie.

Wywołanie: użytkownik mówi coś w stylu "zapisz ten ekran jako wzorzec
`<nazwa>`" albo "dodaj to do biblioteki jako `<nazwa>`". To wywołuje skill,
który przeprowadza 7-krokowy atomowy zapis.

### Przy tagowaniu
Wybieraj tagi WYŁĄCZNIE z `TAGS.md`. Jeśli żaden istniejący tag nie pasuje,
zaproponuj użytkownikowi nowy wpis do listy i poczekaj na akceptację PRZED
zapisem wzorca. Nie dodawaj tagów do `TAGS.md` bez zgody użytkownika.

### Kiedy tworzyć kompozycję
Tylko jeśli ten sam zestaw 3+ sekcji występuje razem w 2+ projektach.
Do tego czasu `compositions/` pozostaje pusty. Kompozycja to
`recipe.md` + `preview.png`, nie duplikat sekcji.

## Zobacz także
- Specyfikacja: `C:/Users/getrk/Projects/pattern-library/docs/specs/2026-04-07-pattern-library-design.md`
```

- [ ] **Step 3.2: Verify file exists and has non-trivial content**

Run:
```bash
wc -l "C:/Users/getrk/patterns/README.md"
```
Expected: at least 60 lines (plik ma być pełny — nie pusty stub).

### Task 4: Write patterns/TAGS.md (controlled tag list)

**Files:**
- Create: `C:/Users/getrk/patterns/TAGS.md`

- [ ] **Step 4.1: Write the tag list**

Write to `C:/Users/getrk/patterns/TAGS.md`:

```markdown
# Kontrolowana lista tagów

Ten plik definiuje pełny zestaw tagów dozwolonych w wzorcach. Każdy wzorzec
zapisany przez skill `save-pattern` musi mieć tylko tagi z tej listy.

## Jak dodać nowy tag
Tag dodaje się tylko za zgodą użytkownika podczas działania skill'a
`save-pattern`. Claude proponuje nowy tag w kontekście zapisu konkretnego
wzorca, użytkownik akceptuje lub odrzuca. Nie edytuj tego pliku ręcznie
bez dobrego powodu.

## Tagi

### Nawigacja
- `navigation` — dowolna forma nawigacji głównej
- `tabs` — tab bar (dolny lub górny)
- `drawer` — menu wysuwane z boku
- `header` — górny pasek (z tytułem, akcjami)
- `breadcrumbs` — ścieżka nawigacyjna

### Struktura treści
- `list` — lista pionowa
- `grid` — siatka
- `card-list` — lista kart
- `feature-showcase` — prezentacja funkcji/zalet
- `hero` — sekcja hero (duży nagłówek z CTA)
- `empty-state` — stan pusty (brak danych)

### Interakcja
- `form` — formularz
- `search` — pole wyszukiwania
- `filters` — chipsy lub dropdowny filtrów
- `toggles` — przełączniki (typowo dla ekranów ustawień)
- `fab` — floating action button
- `modal` — okno modalne

### Domena
- `settings` — ekran ustawień
- `auth` — logowanie, rejestracja, reset hasła
- `onboarding` — powitanie nowego użytkownika
- `profile` — profil użytkownika
- `notifications` — lista lub ustawienia powiadomień

### Stan aplikacji
- `loading` — stan ładowania
- `error` — stan błędu
- `success` — potwierdzenie powodzenia
```

- [ ] **Step 4.2: Verify file exists**

Run:
```bash
grep -c "^- \`" "C:/Users/getrk/patterns/TAGS.md"
```
Expected: `25` (nav 5 + content 6 + interaction 6 + domain 5 + state 3 = 25). Jeśli wartość różni się, sprawdź czy plik został napisany kompletny.

### Task 5: Write patterns/INDEX.md (empty table)

**Files:**
- Create: `C:/Users/getrk/patterns/INDEX.md`

- [ ] **Step 5.1: Write the empty index table**

Write to `C:/Users/getrk/patterns/INDEX.md`:

```markdown
# Pattern Library Index

Ten plik to płaska tabela wszystkich wzorców w bibliotece. Używany przez
Claude'a w Bramce 1 pipeline'u wyszukiwania (grep po tagach). Skill
`save-pattern` dodaje do tego pliku jeden wiersz przy każdym nowym wzorcu.

**Nie edytuj ręcznie** — wpisy dodaje i usuwa skill `save-pattern`.

| Nazwa | Kategoria | Tagi | Ścieżka | Opis |
|---|---|---|---|---|
<!-- Wzorce dodawane poniżej przez skill save-pattern -->
```

- [ ] **Step 5.2: Verify file exists and has the comment marker**

Run:
```bash
grep -c "Wzorce dodawane poniżej" "C:/Users/getrk/patterns/INDEX.md"
```
Expected: `1` (marker musi być obecny — skill `save-pattern` później go używa jako punktu wstawienia nowych wierszy).

### Task 6: Final scaffold verification

- [ ] **Step 6.1: Verify all library files exist**

Run:
```bash
ls "C:/Users/getrk/patterns/README.md" "C:/Users/getrk/patterns/INDEX.md" "C:/Users/getrk/patterns/TAGS.md" && find "C:/Users/getrk/patterns/sections" -type d | wc -l
```
Expected: trzy pliki listed, wartość `7` (sections + 6 podkatalogów).

**Nota (nie jest to krok do wykonania):** Biblioteka `C:/Users/getrk/patterns/` **nie** jest inicjalizowana jako git repo w v1 (patrz spec sekcja 12 — cross-project sync jest w backlogu). Katalogi i pliki są po prostu plikami na dysku. **Nie** uruchamiaj `git init` w `C:/Users/getrk/patterns/`. Meta-projekt został już zacommitowany w Task 1 i Chunk 1 nie ma więcej zmian do zapisania — przejdź do Chunk 2.

---

## Chunk 2: save-pattern Skill

Cel chunka: napisać kompletny skill `save-pattern` w `C:/Users/getrk/.claude/skills/save-pattern.md` zgodny z istniejącą konwencją (single-file, format jak `deploy.md`). Skill musi być wystarczająco szczegółowy, żeby Claude mógł go bez problemu wykonać w sesji, w której użytkownik ma otwarty Pencil z aktywnym `.pen`.

### Task 7: Pre-discovery and write save-pattern.md skill file

**Files:**
- Create: `C:/Users/getrk/Projects/pattern-library/docs/plans/pencil-mcp-notes.md` (scratch notes z pre-discovery)
- Create: `C:/Users/getrk/.claude/skills/save-pattern.md`

**Dlaczego pre-discovery jest obowiązkowe:** Skill `save-pattern` wywołuje
4 narzędzia Pencil MCP (`get_editor_state`, `open_document`, `batch_design`,
`export_nodes`), z których 3 mają nieoczywiste szczegóły:
- Czy `open_document(<nowa_ścieżka>)` faktycznie tworzy plik na dysku?
- Jak Pencil persyst dokumentu — czy jest osobna operacja `save`?
- Czy `batch_design` z operacją `Copy` działa cross-document?
- Jaki jest dokładny kształt argumentów `export_nodes`?

Zamiast wstawiać hedge'i typu "sprawdź guidelines jeśli potrzebujesz",
ten task wymusza pre-discovery JEDEN raz, dokumentuje wynik w
`pencil-mcp-notes.md`, a potem wkleja konkretne kształty do skill'a.
Po skończeniu Task 7 plik `save-pattern.md` nie zawiera żadnych
markerów `<<PENCIL_SHAPE: ...>>`.

**Wymaganie:** Pencil musi być uruchomiony i podłączony do VS Code (lub
innego zewnętrznego edytora obsługiwanego przez Pencil MCP). Bez tego
`get_guidelines` zwraca `failed to connect to running Pencil app` i
nie da się wykonać Task 7.

- [ ] **Step 7.0a: Sprawdź, że Pencil MCP odpowiada**

Wywołaj testowo:
```
mcp__pencil__get_guidelines()
```
Expected: zwraca listę dostępnych guides/styles. Jeśli błąd typu
`failed to connect to running Pencil app`, oznacza to że Pencil nie
jest uruchomiony. Poproś użytkownika, żeby go uruchomił, albo odłóż
Task 7 do momentu gdy Pencil będzie dostępny.

- [ ] **Step 7.0b: Zidentyfikuj guides/styles dotyczące krytycznych operacji**

Z wyniku Step 7.0a wybierz wpisy odnoszące się do:
1. `batch_design` lub operacja `Copy` (cross-document)
2. `export_nodes`
3. `open_document` (zwłaszcza tworzenie nowego pliku i zapis na dysk)
4. `get_editor_state` i `batch_get` (struktura selection, root node)

Jeśli któraś operacja **nie ma** dedykowanego guide'a, użyj jej opisu z
narzędzia MCP (`<system-reminder>` z opisem narzędzi Pencil MCP) jako
źródła prawdy i zapisz w notatkach jako "no guide, tool description only".

- [ ] **Step 7.0c: Załaduj treść guidelines**

Dla każdego zidentyfikowanego guide'a:
```
mcp__pencil__get_guidelines(category, name)
```
i jeśli zwraca prośbę o dodatkowe `params` — wywołaj jeszcze raz z
podanymi parametrami:
```
mcp__pencil__get_guidelines(category, name, params)
```

Zapamiętaj wyniki w pamięci do następnego kroku.

- [ ] **Step 7.0d: Udokumentuj findingi w pencil-mcp-notes.md**

Write to `C:/Users/getrk/Projects/pattern-library/docs/plans/pencil-mcp-notes.md`:

```markdown
# Pencil MCP — pre-discovery notes (save-pattern skill)

Notatki sporządzone przed napisaniem skill'a `save-pattern`. Po użyciu
można zostać dla referencji albo skasować — skill nie zależy od tego
pliku w runtime, kształty są wklejone bezpośrednio do `.claude/skills/save-pattern.md`.

## batch_design z operacją Copy
**Cross-document support:** YES / NO
**Dokładna składnia (single-document):**
[wklej z get_guidelines]

**Dokładna składnia (cross-document) lub fallback:**
[wklej z get_guidelines lub opisz fallback batch_get + I(...)]

**Przykładowe wywołanie kompletne:**
[wklej]

## export_nodes
**Wymagane argumenty:**
[lista]

**Opcjonalne argumenty:**
[lista]

**Czy outputPath zapisuje na dysk?**
[YES/NO + jak]

**Przykładowe wywołanie kompletne (PNG, width 800, do podanej ścieżki):**
[wklej]

## open_document
**Zachowanie gdy ścieżka NIE istnieje:**
[YES tworzy nowy / NO błąd / inne]

**Zachowanie gdy ścieżka istnieje:**
[opis]

**Czy 'new' robi inną rzecz niż konkretna ścieżka nieistniejąca?**
[YES/NO + jak]

**Mechanizm zapisu na dysk po mutacji:**
[opisz konkretnie — czy auto-save? czy trzeba osobno wywołać save? czy zapis przy zamknięciu?]

## get_editor_state + batch_get
**Struktura selection (gdy zaznaczony jest jeden węzeł):**
[wklej fragment]

**Jak uzyskać ID root node'a aktywnego dokumentu:**
[opisz]

**Jak batch_get używa się do uzyskania ID konkretnego węzła z .pen pliku:**
[opisz]
```

- [ ] **Step 7.0e: Zweryfikuj, że żadne pole notes nie zostało puste**

Run:
```bash
grep -c "\[wklej\|\[opisz\|\[lista\|\[YES/NO" "C:/Users/getrk/Projects/pattern-library/docs/plans/pencil-mcp-notes.md"
```
Expected: `0`. Jeśli >0 — wróć do Step 7.0c i uzupełnij brakujące pola
zanim zaczniesz pisać skill. **Nie wolno przejść do Step 7.1 z
niedopełnionymi notes.**

- [ ] **Step 7.1: Verify existing skills directory**

Run:
```bash
ls -la "C:/Users/getrk/.claude/skills/"
```
Expected: istnieje co najmniej `deploy.md` — potwierdzenie że folder jest właściwym miejscem dla user-skilli.

- [ ] **Step 7.2: Write the skill file (z konkretem zamiast markerów)**

Write to `C:/Users/getrk/.claude/skills/save-pattern.md`. Skopiuj
poniższy szablon **i zastąp wszystkie markery `<<PENCIL_SHAPE: ...>>`**
konkretnymi wywołaniami narzędzi Pencil MCP, korzystając z notatek
z `pencil-mcp-notes.md` (sekcje "batch_design Copy", "export_nodes",
"open_document"). Po zapisie nie może zostać żaden marker — Step 7.5
to weryfikuje.

Szablon do skopiowania i wypełnienia:

```markdown
# Skill: save-pattern

Atomowo zapisuje wzorzec (sekcję UI) do biblioteki `C:/Users/getrk/patterns/`.
Jeden wzorzec = katalog z czterema plikami: `design.json`, `preview.png`,
`implementation.tsx`, `README.md`. Aktualizuje też `INDEX.md`. Cały zapis
jest atomowy — albo powstają wszystkie pliki i wpis w indeksie, albo żaden.

## Trigger

Gdy użytkownik mówi:
- "zapisz ten ekran jako wzorzec `<nazwa>`"
- "dodaj to do biblioteki jako `<nazwa>`"
- "zachowaj ten blok jako wzorzec"
- lub wywołuje `/save-pattern`

## Pre-flight checks (zanim zaczniesz cokolwiek zapisywać)

1. Sprawdź, że istnieje aktywny dokument w Pencilu:
   - Wywołaj `mcp__pencil__get_editor_state({ include_schema: false })`
   - Jeśli nie ma aktywnego dokumentu, spytaj użytkownika, który `.pen`
     otworzyć, i otwórz go przez `mcp__pencil__open_document`. Jeśli
     użytkownik odmówi — **abort**, nie zapisuj niczego.

2. Sprawdź, że jest zaznaczony węzeł (źródło wzorca):
   - Z `get_editor_state` odczytaj `selection`. Jeśli puste, spytaj
     użytkownika, który węzeł ma być źródłem wzorca. Jeśli nie da się
     ustalić — **abort**.

3. Sprawdź, że biblioteka istnieje:
   - `ls C:/Users/getrk/patterns/README.md` — musi istnieć. Jeśli nie,
     **abort** z komunikatem: "Biblioteka nieznaleziona w
     C:/Users/getrk/patterns/. Zainicjalizuj bibliotekę najpierw
     (patrz docs/plans/2026-04-07-pattern-library-implementation.md)."

## Workflow

Workflow ma 7 głównych kroków zgodnych z sekcją 5.2 specyfikacji, plus
nienumerowany raport sukcesu na końcu. Kroki są zaprojektowane atomowo:
jeśli którykolwiek zawiedzie po Kroku 2, wykonaj Rollback (patrz sekcja
niżej) — biblioteka nigdy nie może pozostać w stanie pośrednim.

### Krok 1: Zbierz i zwaliduj dane od użytkownika

Zbierz od użytkownika:
- **Nazwa wzorca** (kebab-case, np. `bottom-tab-5`). Jeśli użytkownik
  nie podał w wywołaniu, zaproponuj na podstawie zaznaczonego węzła i
  poproś o akceptację.
- **Kategoria** — jedna z: `navigation`, `hero`, `content`, `forms`,
  `footer`, `empty-states`. Zaproponuj na podstawie charakteru zaznaczenia,
  poproś o akceptację. Użyj `AskUserQuestion` z multiple-choice.
- **Ścieżka do pliku `.tsx`** implementującego ten wzorzec. Jeśli w
  kontekście konwersacji widać konkretny plik, zaproponuj go. W
  przeciwnym razie poproś użytkownika. Zweryfikuj, że plik istnieje
  przez `Read` — jeśli nie istnieje, **abort** (nie twórz wzorca bez
  realnego kodu).
- **Krótki opis** (1 zdanie, do INDEX.md). Zaproponuj na podstawie
  designu + kodu, poproś o akceptację.

**Zwaliduj tagi (wszystko jeszcze w pamięci — żadnych zmian na dysku):**
1. Przeczytaj `C:/Users/getrk/patterns/TAGS.md`.
2. Na podstawie kodu, designu i opisu zaproponuj 2-5 tagów.
3. Zwaliduj każdy zaproponowany tag — musi znajdować się w liście w
   `TAGS.md`. Wyszukaj dokładne dopasowanie wzorca `` - `<tag>` ``.
4. Jeśli któryś tag nie jest w liście:
   - Pokaż użytkownikowi, który tag próbujesz dodać
   - Spytaj, czy chce dodać ten tag do `TAGS.md` (z opisem, w jakiej kategorii)
   - **WAŻNE:** Nawet jeśli użytkownik się zgodzi, NIE zapisuj jeszcze
     `TAGS.md`. Tylko zapamiętaj decyzję w pamięci jako `PENDING_NEW_TAG`
     — plik `TAGS.md` zostanie zapisany dopiero w Kroku 7, razem z
     `INDEX.md`, jako atomowa operacja. Jeśli użytkownik odmówi — zastąp
     tagiem z listy albo **abort**.
5. Pokaż użytkownikowi finalne tagi i poproś o akceptację.

**Sprawdź konflikt nazwy (jeszcze bez mkdir):**
```bash
test -d "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>" && echo "CONFLICT" || echo "OK"
```
Jeśli `CONFLICT`, spytaj użytkownika przez `AskUserQuestion`:
- Nadpisać istniejący
- Podać inną nazwę (wróć do zbierania danych)
- Anulować zapis

Jeśli anulują lub nie wybiorą innej nazwy — **abort**, nic nie zostało
jeszcze zapisane.

### Krok 2: Utwórz katalog docelowy (pierwszy write na dysk)

**Od tego momentu, jeśli którykolwiek krok zawiedzie, wykonaj Rollback
— patrz sekcja "Rollback".**

Utwórz katalog docelowy:
```bash
mkdir -p "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>"
```

Zweryfikuj:
```bash
test -d "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>" && echo "OK" || echo "FAIL"
```
Jeśli `FAIL` — **abort** (nie ma jeszcze czego rollbackować, ale coś jest
zasadniczo nie tak z uprawnieniami lub ścieżką).

### Krok 3: Zapisz design.json (JSON snapshot drzewa węzłów)

**Uwaga architektoniczna:** Pencil MCP nie wspiera tworzenia standalone
`.pen` plików pod dowolną ścieżką (`batch_design` operuje na aktywnym
dokumencie; `open_document` tylko otwiera istniejące pliki). Dlatego
wzorzec zapisuje się jako **JSON snapshot drzewa węzłów** — pełny wynik
`batch_get` zapisany jako `design.json`. Przy użyciu wzorca JSON jest
deserializowany do sekwencji `batch_design I(...)` wstawiającej węzły
do docelowego dokumentu.

Szczegóły kształtów wywołań: `docs/plans/pencil-mcp-notes.md` (sekcje
"batch_get" i "batch_design").

1. Z `get_editor_state()` odczytaj:
   - `SOURCE_PEN_PATH` — ścieżkę aktywnego `.pen`
   - `SOURCE_NODE_ID` — ID zaznaczonego węzła
   Jeśli nie ma zaznaczenia, jesteś poza pre-flight checks — to jest
   bug, abort z rollbackiem.

2. Odczytaj pełne drzewo węzła:
   ```
   mcp__pencil__batch_get({
     filePath: "<SOURCE_PEN_PATH>",
     nodeIds: ["<SOURCE_NODE_ID>"],
     readDepth: 999,
     resolveInstances: true,
     includePathGeometry: true
   })
   ```

3. Zserializuj wynik jako pretty-printed JSON i zapisz:
   ```
   Write "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/design.json"
   ```
   Zawartość: stringify wyniku `batch_get` z indentacją 2 spacji, żeby
   był czytelny ludzkim okiem (do debugowania).

4. Zweryfikuj, że plik istnieje i ma sensowny rozmiar:
   ```bash
   test -f "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/design.json" && wc -c "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/design.json" || echo "MISSING"
   ```
   Jeśli `MISSING` lub rozmiar < 100 bajtów (pusty node tree) — **ROLLBACK**.

### Krok 4: Wyeksportuj preview.png

Eksport bierze węzeł prosto ze źródłowego `.pen` (aktywnego dokumentu), a
nie z `design.json`. Ze schematu `export_nodes` wiemy, że plik wyjściowy
nazywa się `<nodeId>.<ext>` w `outputDir` — skill musi go przemianować
na `preview.png`.

1. Wyeksportuj węzeł do katalogu wzorca:
   ```
   mcp__pencil__export_nodes({
     filePath: "<SOURCE_PEN_PATH>",
     outputDir: "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>",
     nodeIds: ["<SOURCE_NODE_ID>"],
     format: "png",
     scale: 2
   })
   ```
   To tworzy `<SOURCE_NODE_ID>.png` w `outputDir`.

2. Przemianuj plik na `preview.png`:
   ```bash
   mv "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/<SOURCE_NODE_ID>.png" \
      "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/preview.png"
   ```

3. Zweryfikuj, że plik istnieje i jest realną grafiką:
   ```bash
   test -f "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/preview.png" && SIZE=$(wc -c < "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/preview.png") && test "$SIZE" -ge 1024 && echo "OK ($SIZE bytes)" || echo "FAIL"
   ```
   Jeśli `FAIL` — **ROLLBACK** (pusty PNG zwykle ma <100 bajtów, realny
   render powinien przekraczać 1 KB).

### Krok 5: Skopiuj implementation.tsx z nagłówkiem

1. Odczytaj oryginalny plik `.tsx` użytkownika przez `Read`. Nazwij
   zawartość `RAW_CODE`. Policz linie: `RAW_LINE_COUNT = wc -l`.

2. Zidentyfikuj **zależności** do nagłówka, czytając linie `import`
   na początku pliku:
   - Nazwane pakiety npm (np. `expo-router`, `gluestack-ui`)
   - Pakiety platform (`react-native`, `react`)
   - Custom hooki z projektu źródłowego (np. `useColors`, `useAuthStore`)

3. Zidentyfikuj **punkty adaptacji** — miejsca, które na pewno trzeba
   zmienić przy kopiowaniu wzorca do innego projektu. Typowe kandydaty:
   - Linie importów wskazujące na konkretne moduły projektu źródłowego
     (np. `import { useColors } from '@/hooks/useColors'`)
   - Hardkodowane teksty w języku źródłowym (np. polskie etykiety)
   - Nazwy tras / typy domeny (np. `Task`, `Product`)
   - Zewnętrzne zależności specyficzne dla projektu

   Dla każdego punktu adaptacji zapisz **numer linii w `RAW_CODE`**
   (numerowanie od 1, nie od pliku z nagłówkiem!).

4. Wygeneruj nagłówek-komentarz, uzupełniając wszystkie pola:

   ```tsx
   // ════════════════════════════════════════════════════════════
   // Pattern: sections/<kategoria>/<nazwa>
   // Source: <source_project> (<relative_source_path>)
   // Saved: <YYYY-MM-DD>
   //
   // Dependencies:
   //   - <pakiet_1> (<czego_używa>)
   //   - <pakiet_2> (<czego_używa>)
   //   - hook: <custom_hook> (<do_czego>)
   //
   // Adaptation points (numeracja od pierwszej linii kodu pod tym
   // nagłówkiem, nie od początku pliku):
   //   L<n>  — <co zmienić>
   //   L<n>  — <co zmienić>
   // ════════════════════════════════════════════════════════════

   ```

   **Krytyczne:** Numery `L<n>` to numery linii w `RAW_CODE` (od 1),
   NIE w pliku z dodanym nagłówkiem. Nagłówek ma stałą długość ~17 linii;
   jeśli kiedyś trzeba przeliczyć na "pełny plik", użytkownik może dodać
   tę stałą.

5. Skomponuj finalną zawartość: `<nagłówek>` + `\n` + `RAW_CODE`.

6. Zapisz przez `Write`:
   ```
   Write C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/implementation.tsx
   ```

7. Zweryfikuj — plik powinien mieć **co najmniej `RAW_LINE_COUNT + 17`** linii:
   ```bash
   wc -l "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/implementation.tsx"
   ```
   Jeśli mniej — coś poszło nie tak (zapisany niekompletny plik) → **ROLLBACK**.

### Krok 6: Wygeneruj i zapisz README.md

1. Przygotuj draft README z frontmatter + sekcje opisowe. Szablon:

   ```markdown
   ---
   name: <nazwa>
   category: sections/<kategoria>
   tags: [<tag1>, <tag2>, ...]
   created: <YYYY-MM-DD>
   source_project: <nazwa_projektu>
   source_file: <relatywna_ścieżka_do_oryginału>
   dependencies:
     - <pakiet_1>
     - <pakiet_2>
     - hook: <custom_hook>
   ---

   # <Czytelna nazwa wzorca>

   ## Co to jest
   <1-3 zdania opisujące co to jest i z czego się składa. Konkretnie —
   nie "ładny ekran", tylko "ekran z headerem, listą z filtrami i FAB".>

   ## Kiedy używać
   - <kryterium 1>
   - <kryterium 2>
   - <kryterium 3>

   ## Kiedy NIE używać
   - <kryterium 1>
   - <kryterium 2>

   ## Zależności do zainstalowania
   - <pakiet>
   - <pakiet>

   ## Punkty adaptacji (co zmienić przy kopiowaniu)
   - **L<n>** — <co zmienić>
   - **L<n>** — <co zmienić>
   ```

2. Pokaż draft użytkownikowi i poproś o akceptację (UI draftu:
   przytoczony Markdown). Maksymalnie 3 iteracje poprawek. Jeśli po 3
   iteracjach użytkownik nadal nie jest zadowolony — ROLLBACK.

3. Po akceptacji zapisz:
   ```
   Write C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/README.md
   ```

### Krok 7: Zaktualizuj INDEX.md (i opcjonalnie TAGS.md)

To jest ostatni numerowany krok. Wszystkie poprzednie artefakty są już
na dysku — teraz dopisujemy wpisy do indeksu i (jeśli dotyczy) zatwierdzamy
nowy tag w TAGS.md jako jedna atomowa operacja.

1. **Jeśli `PENDING_NEW_TAG` jest ustawione w pamięci** (Krok 1 ustalił,
   że użytkownik zaakceptował dodanie nowego tagu) — zaktualizuj `TAGS.md`:
   - `Read C:/Users/getrk/patterns/TAGS.md`
   - Użyj `Edit` żeby dodać wiersz `- \`<nowy_tag>\` — <opis>` w
     odpowiedniej sekcji semantycznej (np. pod `## Domena`)
   - Zweryfikuj:
     ```bash
     grep -c "^- \`<nowy_tag>\`" "C:/Users/getrk/patterns/TAGS.md"
     ```
     Expected: `1`. Jeśli `0` — **ROLLBACK** (z dodatkowym revertem
     TAGS.md, patrz sekcja Rollback).

2. Przeczytaj `C:/Users/getrk/patterns/INDEX.md`.

3. Przygotuj nowy wiersz w formacie:
   ```
   | <nazwa> | <kategoria> | <tag1>, <tag2> | sections/<kategoria>/<nazwa> | <opis> |
   ```

4. Użyj `Edit` do wstawienia wiersza **przed** markerem
   `<!-- Wzorce dodawane poniżej przez skill save-pattern -->`. Staraj
   się utrzymać alfabetyczną kolejność wierszy w ramach tej samej kategorii.

5. Potwierdź zapis:
   ```bash
   grep -c "| <nazwa> |" "C:/Users/getrk/patterns/INDEX.md"
   ```
   Expected: `1`. Jeśli `0` — **ROLLBACK** (z revertami INDEX i TAGS).

### Raport sukcesu (post-workflow, nienumerowany)

Poinformuj użytkownika zwięźle:
```
Zapisano wzorzec sections/<kategoria>/<nazwa>/
- design.json (<rozmiar>)
- preview.png (<rozmiar>)
- implementation.tsx (<liczba linii> linii, w tym 17 linii nagłówka)
- README.md

INDEX.md: +1 wpis (<N> wzorców total)
TAGS.md: <"+1 nowy tag" jeśli PENDING_NEW_TAG, "bez zmian" w przeciwnym razie>
```

## Rollback

Jeśli którykolwiek krok po Kroku 2 (kiedy utworzyliśmy katalog docelowy)
zawiedzie, ZAWSZE wykonaj rollback w odwrotnej kolejności do tego, co
zostało zapisane:

1. **Jeśli Krok 7 częściowo wykonał się i zmodyfikowałeś INDEX.md**
   (już dodałeś nowy wiersz), cofnij to:
   - `Read INDEX.md`, `Edit` aby usunąć dodany wiersz
   - Zweryfikuj: `grep -c "<nazwa>" INDEX.md` powinno zwrócić `0`

2. **Jeśli Krok 7 zmodyfikował TAGS.md** (dodał nowy tag), cofnij to:
   - `Read TAGS.md`, `Edit` aby usunąć dodany wiersz
   - Zweryfikuj: `grep -c "<nowy_tag>" TAGS.md` powinno zwrócić `0`

3. **Usuń katalog docelowy** (zawiera design.json, preview.png,
   implementation.tsx, README.md — to wszystko od Kroków 2-6):
   ```bash
   rm -rf "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>"
   ```

4. **Poinformuj użytkownika** konkretnie, który krok zawiódł i dlaczego.
   Powiedz wyraźnie, że biblioteka pozostała w stanie spójnym.

**Zasada:** lepiej brak wzorca niż pół-wzorzec. Biblioteka musi pozostać
spójna. **PENDING_NEW_TAG** jest celowo trzymany tylko w pamięci do
Kroku 7 — dzięki temu rollback przed Krokiem 7 nie wymaga dotykania
TAGS.md w ogóle.

## Co może pójść nie tak (typowe błędy i jak reagować)

| Problem | Reakcja |
|---|---|
| Brak aktywnego `.pen` | Pre-flight check — spytaj i otwórz, albo abort |
| Brak zaznaczenia | Pre-flight check — spytaj, co ma być źródłem |
| `.tsx` nie istnieje | Abort w Kroku 1 (walidacja inputu) |
| Tag spoza TAGS.md bez zgody na dodanie | Abort albo zastąp znanym tagiem |
| Konflikt nazwy wzorca | Krok 1 — spytaj: nadpisz / inna nazwa / anuluj |
| `mkdir` w Kroku 2 nie działa | Abort (brak uprawnień / zła ścieżka) |
| Pencil MCP zwraca błąd w Kroku 3 (copy/persist) | ROLLBACK, raport błędu |
| `export_nodes` w Kroku 4 zwraca błąd | ROLLBACK, raport błędu |
| Eksport w Kroku 4 daje pusty PNG (<1 KB) | ROLLBACK, raport błędu |
| Użytkownik odrzuca draft README 3× w Kroku 6 | ROLLBACK, raport |
| `Edit` na INDEX.md lub TAGS.md w Kroku 7 zawiedzie | ROLLBACK z revertem już zmodyfikowanego pliku |

## Ważne uwagi

- **Nigdy nie modyfikuj plików w istniejącym wzorcu bez wyraźnej prośby
  użytkownika o edycję.** Skill jest do zapisu nowych wzorców, nie do
  aktualizacji istniejących.
- **Numery linii w nagłówku `implementation.tsx` są liczone od pierwszej
  linii kodu pod zamknięciem nagłówka-komentarza** (nie od początku pliku).
- **Nie twórz wzorca z kodu, którego nie można odtworzyć** — jeśli
  oryginalny `.tsx` odwołuje się do rzeczy spoza projektu źródłowego
  (np. globalny helper w innym repo), zapisz to wyraźnie w sekcji
  "Punkty adaptacji".
- **Claude nie dodaje sam nowych tagów do TAGS.md bez zgody użytkownika.**
  Kontrolowana lista ma być kontrolowana.
- **Komunikacja z użytkownikiem jest w języku polskim** (domena
  użytkownika i projektu HouseKeep).
```

- [ ] **Step 7.3: Verify the skill file exists and has expected sections**

Wykonaj każde sprawdzenie pojedynczo (zamiast pętli — bezpieczniejsze
na różnych powłokach):

```bash
grep -c "^# Skill: save-pattern" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "^## Trigger" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "^## Pre-flight checks" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "^## Workflow$" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "^### Krok 1: Zbierz i zwaliduj dane" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "^### Krok 7: Zaktualizuj INDEX.md" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "^## Rollback" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "^## Co może pójść nie tak" "C:/Users/getrk/.claude/skills/save-pattern.md"
```

Expected: każde polecenie zwraca `1`. Jeśli któraś zwraca `0` —
brakuje sekcji, popraw.

- [ ] **Step 7.4: Verify the skill file is in the right place**

Run:
```bash
ls -la "C:/Users/getrk/.claude/skills/"
```
Expected: widać co najmniej `deploy.md` i `save-pattern.md`.

- [ ] **Step 7.5: Verify no hand-waved Pencil MCP markers remain**

Po pre-discovery (Task 7 początek) wszystkie `<<PENCIL_SHAPE: ...>>`
markery w skill'u musiały zostać zastąpione konkretnymi wywołaniami
narzędzi z `pencil-mcp-notes.md`. Sprawdź, że żaden marker nie pozostał:

```bash
grep -c "<<PENCIL_SHAPE" "C:/Users/getrk/.claude/skills/save-pattern.md"
grep -c "sprawdź przez get_guidelines\|jeśli potrzebujesz potwierdzenia" "C:/Users/getrk/.claude/skills/save-pattern.md"
```

Expected: oba polecenia zwracają `0`. Jeśli którekolwiek zwraca >0 —
wróć do Steps 7.0a-7.0e (pre-discovery) i Step 7.2, wklej brakujące
kształty.

### Task 8: Concrete skill verification (zastępuje wcześniejszy "dry-run")

Zamiast mentalnej symulacji ten task wykonuje konkretne sprawdzenia
nad zapisanym plikiem skill'a. Każdy step jest weryfikowalny — albo
output pasuje, albo nie.

- [ ] **Step 8.1: Sprawdź, że każde wywołanie Pencil MCP w skill'u ma konkretne argumenty**

Run:
```bash
grep -n "mcp__pencil__" "C:/Users/getrk/.claude/skills/save-pattern.md"
```

Dla każdego wystąpienia ręcznie sprawdź:
- Argumenty są zapisane jak w pencil-mcp-notes.md (konkretne nazwy pól, konkretne wartości lub zmienne runtime jak `<nazwa>`, `<kategoria>`, `SOURCE_NODE_ID`, `TARGET_ROOT_ID`)
- Brak placeholderów typu `<?>`, `[paste]`, `(do uzupełnienia)`, `<<PENCIL_SHAPE`
- Brak fraz typu "sprawdź guidelines" w okolicy wywołania

Jeśli któreś wywołanie ma niejasność — wróć do Step 7.2 i wypełnij
korzystając z `pencil-mcp-notes.md`.

- [ ] **Step 8.2: Sprawdź mapowanie kroków skill'a na 7 kroków specyfikacji**

Dla każdego z 7 kroków specyfikacji (sekcja 5.2) sprawdź, że istnieje
odpowiadający `### Krok N:` w skill'u:

```bash
grep -E "^### Krok [1-7]: " "C:/Users/getrk/.claude/skills/save-pattern.md"
```

Expected: dokładnie 7 dopasowań, w kolejności:
- `### Krok 1: Zbierz i zwaliduj dane od użytkownika`
- `### Krok 2: Utwórz katalog docelowy`
- `### Krok 3: Zapisz design.json`
- `### Krok 4: Wyeksportuj preview.png`
- `### Krok 5: Skopiuj implementation.tsx z nagłówkiem`
- `### Krok 6: Wygeneruj i zapisz README.md`
- `### Krok 7: Zaktualizuj INDEX.md`

Jeśli liczba lub nazwy się różnią — popraw skill.

- [ ] **Step 8.3: Sprawdź, że Rollback jawnie obsługuje TAGS.md i INDEX.md**

Run:
```bash
grep -A 20 "^## Rollback" "C:/Users/getrk/.claude/skills/save-pattern.md" | grep -c "TAGS.md\|INDEX.md"
```
Expected: `>= 2` (obie nazwy plików muszą wystąpić w sekcji Rollback).
Jeśli mniej — sekcja Rollback nie obsługuje wszystkich możliwych
artefaktów Kroku 7. Popraw.

- [ ] **Step 8.4: Sprawdź, że nie ma wspomnienia o Krok 8 lub Krok 9 (relikty starszej struktury)**

Run:
```bash
grep -c "Krok 8\|Krok 9" "C:/Users/getrk/.claude/skills/save-pattern.md"
```
Expected: `0`. Workflow ma 7 kroków + raport sukcesu. Jeśli >0 —
gdzieś została stara numeracja, popraw.

**Nota: Chunk 2 nie commituje niczego do meta-projektu.** Skill `save-pattern.md`
żyje w `C:/Users/getrk/.claude/skills/` (poza repo pattern-library), a
`pencil-mcp-notes.md` żyje w meta-projekcie ale jest scratch notes —
możesz go zacommitować w Chunku 3 razem z innymi poprawkami planu lub
zostawić nieśledzony w git. Wybór należy do Ciebie. Przejdź do Chunk 3.

---

## Chunk 3: First Real Pattern (End-to-End Verification)

Cel chunka: potwierdzić, że cały system działa, zapisując PIERWSZY prawdziwy wzorzec z HouseKeep (dolny tab bar nawigacji).

**Ten chunk jest INHERENTNIE INTERAKTYWNY.** Wymaga:
- Otwartego Pencila z aktywnym `.pen` zawierającym design docelowego wzorca
- Użytkownika w pętli (do akceptacji nazwy, kategorii, tagów, draftu README)
- Nowej sesji Claude Code jeśli skill `save-pattern` został dopiero co utworzony (Claude Code ładuje user-skille przy starcie sesji)

Jeśli plan jest wykonywany przez subagent bez dostępu do Pencila lub użytkownika, **ten chunk należy odroczyć do interaktywnej sesji z użytkownikiem** i zakończyć v1 po Chunku 2 z notatką "Chunk 3 pending manual verification".

### Task 10: Prepare the test candidate

**Files:**
- Read: `C:/Users/getrk/Projects/housekeep/mobile/src/app/(app)/_layout.tsx`

- [ ] **Step 10.1: Zweryfikuj, że plik implementacji istnieje**

Run:
```bash
ls "C:/Users/getrk/Projects/housekeep/mobile/src/app/(app)/_layout.tsx"
```
Expected: plik istnieje. Jeśli nie istnieje (struktura HouseKeep różni się), popros użytkownika o podanie aktualnej ścieżki i zapisz ją — użyjesz jej w Kroku 11.

- [ ] **Step 10.2: Zweryfikuj, że istnieje design tab bara w .pen**

Poproś użytkownika, żeby:
1. Otworzył w Pencilu plik HouseKeep zawierający design tab bara
   (jeśli taki istnieje — jeśli nie, ten Task jest niewykonalny w tej
   sesji i powinien być odroczony)
2. Zaznaczył Frame reprezentujący tab bar

Jeśli design nie istnieje w Pencilu (np. HouseKeep był projektowany
poza Pencilem), zamiast dolnego tab bara wybierz inny kandydat, który
ma zarówno design jak i kod. Poproś użytkownika o wskazanie.

### Task 11: Execute save-pattern end-to-end

- [ ] **Step 11.1: Upewnij się, że skill jest załadowany w sesji**

Claude Code ładuje user-skille z `C:/Users/getrk/.claude/skills/` przy
starcie sesji. Jeśli plik `save-pattern.md` został utworzony w tej samej
sesji, w której jesteś teraz, skill **nie jest widoczny** przez `Skill`
tool dopóki sesja nie zostanie zrestartowana.

Sprawdź, czy w liście dostępnych skilli widnieje `save-pattern` (patrz
`<system-reminder>` z listą skilli na starcie sesji). Jeśli nie widać:
1. Zamknij bieżącą sesję Claude Code
2. Otwórz nową sesję w tym samym katalogu
3. Wznów plan od tego kroku

- [ ] **Step 11.2: Wywołaj skill save-pattern**

W sesji z załadowanym skillem powiedz:

> "Użyj skill'a save-pattern: zapisz aktualnie zaznaczony Frame jako wzorzec
> `bottom-tab-5`, kategoria `navigation`, źródło housekeep, plik
> `<rzeczywista ścieżka z Kroku 10.1>`."

- [ ] **Step 11.3: Obserwuj przebieg skill'a i odpowiadaj na pytania**

Skill przeprowadzi Cię przez Pre-flight → 7 kroków workflow. Odpowiadaj
na pytania (akceptacja nazwy, kategorii, tagów, draftu README).

### Task 12: Verify the pattern landed correctly

- [ ] **Step 12.1: Sprawdź, że wszystkie 4 pliki istnieją**

Run:
```bash
ls -la "C:/Users/getrk/patterns/sections/navigation/bottom-tab-5/"
```
Expected: 4 pliki — `design.json`, `preview.png`, `implementation.tsx`, `README.md`.

- [ ] **Step 12.2: Sprawdź, że INDEX został zaktualizowany**

Run:
```bash
grep "bottom-tab-5" "C:/Users/getrk/patterns/INDEX.md"
```
Expected: jeden wiersz z nazwą, kategorią, tagami, ścieżką, opisem.

- [ ] **Step 12.3: Sprawdź frontmatter README**

Run:
```bash
head -12 "C:/Users/getrk/patterns/sections/navigation/bottom-tab-5/README.md"
```
Expected: widzisz `---` na L1, pola `name`, `category`, `tags`, `created`, `source_project`, `source_file`, `dependencies`, zamykające `---`.

- [ ] **Step 12.4: Sprawdź nagłówek implementation.tsx**

Run:
```bash
head -20 "C:/Users/getrk/patterns/sections/navigation/bottom-tab-5/implementation.tsx"
```
Expected: widzisz linię `// Pattern: sections/navigation/bottom-tab-5`,
metadane Source/Saved/Dependencies, sekcję "Adaptation points" z
wylistowanymi linijkami.

- [ ] **Step 12.5: Sprawdź, że preview.png jest realną grafiką**

Run:
```bash
wc -c "C:/Users/getrk/patterns/sections/navigation/bottom-tab-5/preview.png"
```
Expected: pierwsza liczba (rozmiar w bajtach) >= `1024`. Pusty lub
uszkodzony PNG ma zwykle kilkadziesiąt bajtów — prawdziwy render
z Pencila powinien przekraczać 1 KB.

- [ ] **Step 12.6: Sprawdź, że wszystkie tagi w README są w TAGS.md**

Najpierw odczytaj tagi z frontmattera README:
```bash
grep "^tags:" "C:/Users/getrk/patterns/sections/navigation/bottom-tab-5/README.md"
```

Wynik to coś w stylu `tags: [navigation, tabs]`. Wyciągnij nazwy tagów
i dla każdego wykonaj (podmieniając `<tag>` na konkretną nazwę):

```bash
grep -c "^- \`<tag>\`" "C:/Users/getrk/patterns/TAGS.md"
```

Expected: `1` dla każdego wywołania. Jeśli choć jeden tag zwraca `0`,
oznacza to że skill pozwolił na tag spoza listy — bug w Kroku 1
skill'a (walidacja tagów). Zgłoś jako issue i wróć do Chunk 2.

Przykład pełny (dla tagów `navigation` i `tabs`):
```bash
grep -c "^- \`navigation\`" "C:/Users/getrk/patterns/TAGS.md"  # oczekiwane: 1
grep -c "^- \`tabs\`" "C:/Users/getrk/patterns/TAGS.md"        # oczekiwane: 1
```

### Task 13: Test rollback (inject a mid-sequence failure)

Cel: zweryfikować, że rollback sprząta artefakty stworzone **po** utworzeniu
katalogu docelowego. Scenariusze pre-flight abort (brak pliku, brak
zaznaczenia) nie testują rollbacku — w tych przypadkach skill przerywa
ZANIM cokolwiek utworzy, więc nie ma czego sprzątać. Rollback musi być
testowany scenariuszem, który zawodzi po Kroku 4 (mkdir + pierwszy
artefakt na dysku).

- [ ] **Step 13.1: Użyj scenariusza rejection-of-README**

Skill Krok 6 tworzy draft README i czeka na akceptację użytkownika, z
maksymalnie 3 iteracjami. 3× odrzucenie draftu wymusza rollback po
utworzeniu katalogu + `design.json` + `preview.png` + `implementation.tsx`.
To jedyny niezawodny sposób na sprowokowanie rollbacku w sesji
interaktywnej bez modyfikacji skill'a.

- [ ] **Step 13.2: Wywołaj skill save-pattern na NOWYM zaznaczeniu**

Nie używaj tego samego zaznaczenia co w Task 11 — wybierz dowolny inny
Frame w Pencilu (np. inny mały blok w HouseKeep). Wywołaj:

> "Użyj skill'a save-pattern: zapisz zaznaczony Frame jako wzorzec
> `test-rollback`, kategoria `content`, źródło housekeep, plik
> `<dowolna istniejąca ścieżka .tsx z HouseKeep>`."

Gdy skill pokaże draft README (Krok 6), odrzucaj go 3 razy pod rząd z
komunikatem "nie podoba mi się" (bez konstruktywnej poprawki). Po 3.
odrzuceniu skill powinien wykonać rollback.

- [ ] **Step 13.3: Zweryfikuj, że rollback posprzątał**

Run:
```bash
test ! -d "C:/Users/getrk/patterns/sections/content/test-rollback" && echo "OK: directory removed" || echo "FAIL: directory remains"
grep -c "test-rollback" "C:/Users/getrk/patterns/INDEX.md"
grep -c "test-rollback" "C:/Users/getrk/patterns/TAGS.md"
```
Expected:
- Pierwsza linia: `OK: directory removed`
- Druga linia: `0` (brak wpisu w INDEX)
- Trzecia linia: `0` (brak przypadkowo dodanych tagów w TAGS.md)

Jeśli którykolwiek check zawiedzie (katalog zostal, wpis w INDEX dodany,
TAGS.md pollutowany), skill ma błąd w rollbacku — wróć do Chunk 2,
dopracuj sekcję "Rollback" i wykonaj ponownie. Szczególnie sprawdź, czy
skill commituje/zapisuje `TAGS.md` przed rollbackiem — powinien trzymać
proponowane zmiany w pamięci do momentu sukcesu całego workflow.

### Task 14: Final commit (conditional) and summary

Skill `save-pattern.md` żyje w `C:/Users/getrk/.claude/skills/` (poza repo),
a biblioteka w `C:/Users/getrk/patterns/` (też poza repo). **Chunk 3 sam
w sobie nie generuje zmian w meta-projekcie.** Jedynym powodem, dla
którego meta-projekt mógłby mieć delta do zacommitowania, jest sytuacja,
w której podczas Task 8 (concrete skill verification) lub Task 13 (test rollbacku)
znalazłeś bug i poprawiłeś plan lub spec.

- [ ] **Step 14.1: Sprawdź, czy meta-projekt ma zmiany**

Run:
```bash
cd "C:/Users/getrk/Projects/pattern-library" && git status --short
```

- Jeśli output jest PUSTY → **pomiń Step 14.2**, przejdź do Step 14.3.
- Jeśli output zawiera linie ze zmianami → przejdź do Step 14.2.

- [ ] **Step 14.2: Commit poprawek planu/specu (tylko jeśli są zmiany)**

Run:
```bash
cd "C:/Users/getrk/Projects/pattern-library" && git add -A && git commit -m "docs: fixes discovered during pattern-library v1 verification"
```

- [ ] **Step 14.3: Podsumowanie końcowe dla użytkownika**

Napisz krótki raport:
```
Pattern Library v1 — gotowe.

Biblioteka:       C:/Users/getrk/patterns/  (1 wzorzec)
Skill:            C:/Users/getrk/.claude/skills/save-pattern.md
Meta-projekt:     C:/Users/getrk/Projects/pattern-library/  (git repo)

Pierwszy wzorzec: sections/navigation/bottom-tab-5

Następne kroki (nie w v1):
  - Dodanie kolejnych wzorców z HouseKeep (list-with-filters,
    settings-grouped-list, empty-states)
  - Obserwacja, kiedy compositions zacznie mieć sens
  - Ewentualne wersjonowanie TAGS.md
```

---

## Success Criteria (z section 10 specu)

Po zakończeniu wszystkich trzech chunków:

1. Istnieje biblioteka `C:/Users/getrk/patterns/` z kompletną strukturą
   (README, INDEX, TAGS, pustymi katalogami kategorii, pustym
   `compositions/`).
2. Istnieje skill `C:/Users/getrk/.claude/skills/save-pattern.md` z
   kompletnym workflow, pre-flight checks i rollback.
3. Istnieje meta-projekt git repo pod `C:/Users/getrk/Projects/pattern-library/`
   z zacommitowanym spec-em i planem.
4. Co najmniej JEDEN prawdziwy wzorzec z HouseKeep jest zapisany end-to-end
   (wszystkie 4 pliki + wpis w INDEX) i przeszedł weryfikację formatu.
5. Rollback został zweryfikowany na realnym scenariuszu błędu.

Gdy wszystkie 5 punktów jest ✅, v1 jest ukończony i można zamknąć plan.
