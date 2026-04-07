# Pencil MCP — notatki z pre-discovery

Te notatki powstały podczas wykonywania Chunk 2 (save-pattern skill). Zawierają konkretne kształty wywołań narzędzi Pencil MCP i kluczowe ograniczenia, które ujawniły się podczas testów. Skill `save-pattern.md` używa tych findingów bezpośrednio — w skill'u nie ma placeholderów, tylko konkretne wywołania.

**Data discovery:** 2026-04-07
**Źródło:** Odczyt schemy narzędzi via ToolSearch (definitive source) + test kontrolowanych wywołań na aktywnym edytorze.

## batch_design — kluczowe ograniczenie

**Schema:**
```json
{
  "filePath": "string (REQUIRED)",
  "operations": "string (REQUIRED) — operation list w JS-like syntax"
}
```

**Kluczowe odkrycie:** Parametr `filePath` **nie określa pliku wyjściowego** — operacje wykonują się na **aktywnym edytorze**. Test:
- Wywołałem `batch_design(filePath="C:/Users/getrk/test.pen", operations="t=I(document,{type:'text',content:'x'})")`
- Tool zwrócił `Successfully executed all operations` z ID `L70A9`
- `C:/Users/getrk/test.pen` NIE powstał na dysku
- `L70A9` pojawił się jako nowy top-level frame w **aktywnym edytorze** (PaniGinekologPEN.pen)
- Node został następnie skasowany przez `D("L70A9")` (uwaga: D bierze string ID, nie binding)

**Wniosek dla skill'a:** Nie próbuj tworzyć nowych `.pen` plików przez `batch_design`. Traktuj `filePath` jako target dla operacji tylko gdy plik już istnieje i jest aktywny w edytorze.

**Składnia operacji:**
- Insert: `foo=I("parent_id_or_binding", {type, ...})`
- Copy: `baz=C("source_node_id", "parent", {overrides})` — w obrębie JEDNEGO pliku, cross-document nie jest wspierany
- Update: `U(binding+"/child", {...})`
- Replace: `foo2=R("path", {...})`
- Delete: `D("nodeid_string")` — UWAGA: string, nie binding
- `document` to predefiniowany binding = root aktywnego dokumentu
- Max 25 operacji per call
- Binding names unique per call

## export_nodes — konkretny kształt

**Schema:**
```json
{
  "filePath": "string (REQUIRED)",
  "outputDir": "string (REQUIRED)",
  "nodeIds": "string[] (REQUIRED)",
  "format": "png|jpeg|webp|pdf (default: png)",
  "scale": "number (default: 2, max internal res 8192)",
  "quality": "number 1-100 (JPEG/WEBP only)"
}
```

**Kluczowe odkrycie:** Plik wyjściowy nazywa się **`{node_id}.{ext}`** w `outputDir`. Nie ma parametru na wybranie nazwy pliku. Skill MUSI przemianować plik po eksporcie.

**Konkretny przykład dla save-pattern:**
```
mcp__pencil__export_nodes({
  filePath: "<SOURCE_PEN_PATH>",
  outputDir: "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>",
  nodeIds: ["<SOURCE_NODE_ID>"],
  format: "png",
  scale: 2
})
```

Po wywołaniu w katalogu jest `<SOURCE_NODE_ID>.png` — skill robi:
```bash
mv "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/<SOURCE_NODE_ID>.png" \
   "C:/Users/getrk/patterns/sections/<kategoria>/<nazwa>/preview.png"
```

## open_document — tylko do otwierania, nie tworzenia

**Schema:**
```json
{
  "filePathOrTemplate": "string (REQUIRED) — 'new' lub absolutna ścieżka"
}
```

**Kluczowe odkrycie:** `'new'` tworzy pusty dokument w edytorze (nie na dysku pod wybraną ścieżką). Podanie ścieżki OTWIERA istniejący plik. Nie da się stworzyć nowego `.pen` pod wybraną ścieżką via MCP. Nie ma `save_as`. Dlatego `save-pattern` NIE używa `open_document` wcale — zamiast tego korzysta z `batch_get` na aktywnym dokumencie do odczytu węzła i zapisuje JSON.

## batch_get — odczyt drzewa węzłów

**Schema:**
```json
{
  "filePath": "string (REQUIRED)",
  "nodeIds": "string[] (optional) — ID węzłów do odczytu",
  "patterns": "array (optional) — wzorce wyszukiwania",
  "parentId": "string (optional) — scope'owanie do poddrzewa",
  "readDepth": "number (default 1) — głębokość odczytu",
  "searchDepth": "number — głębokość wyszukiwania",
  "resolveInstances": "boolean — rozwijanie component instances",
  "resolveVariables": "boolean — rozwiązywanie zmiennych",
  "includePathGeometry": "boolean — pełna geometria ścieżek SVG"
}
```

**Konkretny przykład dla save-pattern Krok 3:**
```
mcp__pencil__batch_get({
  filePath: "<SOURCE_PEN_PATH>",
  nodeIds: ["<SOURCE_NODE_ID>"],
  readDepth: 999,
  resolveInstances: true,
  includePathGeometry: true
})
```

Wynik to kompletne drzewo węzła wzorca (zamiast skrótu `...` dla children). Ten JSON jest serializowany i zapisywany jako `design.json`.

## get_editor_state — odczyt aktywnego stanu

**Schema:**
```json
{
  "include_schema": "boolean (optional)"
}
```

**Zwraca:** nazwę aktywnego `.pen`, listę top-level nodes z ID i typami, aktualną selekcję. Używane przez skill do ustalenia `SOURCE_PEN_PATH` (aktywny dokument) i `SOURCE_NODE_ID` (zaznaczony węzeł).

**Przykład wyniku (z testów):**
```
## Currently active editor
- `/c:/laragon/www/.../PaniGinekologPEN.pen`

## Document State:
- No nodes are selected.

### Top-Level Nodes (5):
- `j4EOk` (frame): opengraph [user visible]
- `z54kK` (frame): wytyczne [user visible]
...
```

**Uwaga do skill'a:** Ścieżka `.pen` w get_editor_state ma format `/c:/...` (Unix-style z mountpointem), nie `C:/...`. Przy użyciu jako argument innych wywołań Pencil MCP, użyj dokładnie formy zwróconej przez get_editor_state. W kodzie `.tsx`/bash używaj standardowych Windows ścieżek `C:/...`.

## Workflow save-pattern po uwzględnieniu findingów

```
Krok 3: Zapisz design.json
  1. Odczytaj SOURCE_PEN_PATH i SOURCE_NODE_ID z get_editor_state
  2. batch_get z pełną głębokością → JSON drzewo węzła
  3. Write patterns/<kat>/<nazwa>/design.json z tym JSONem

Krok 4: Wyeksportuj preview.png
  1. export_nodes z filePath=SOURCE_PEN_PATH, outputDir=<katalog wzorca>,
     nodeIds=[SOURCE_NODE_ID], format=png, scale=2
  2. mv <SOURCE_NODE_ID>.png preview.png
  3. wc -c preview.png → sprawdź >= 1024
```

## Co zniknęło ze skill'a w wyniku discovery

- Wszystkie `<<PENCIL_SHAPE: ...>>` markery — zastąpione konkretami z tego dokumentu
- Pre-discovery steps 7.0a-7.0e — pre-discovery zostało zrobione przy pierwszym odpaleniu Chunka 2 i jego wynik to ten dokument
- Krok z `open_document('new')` i próbą zapisu pod docelową ścieżką — nie działa, porzucone
- Wymaganie, że `design.pen` jest standalone plikiem — zastąpione przez `design.json` jako JSON sidecar
