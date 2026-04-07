# Session log — 2026-04-07 bootstrap

Pierwsza sesja Pattern Library — od pomysłu do działającej biblioteki z 7 wzorcami i 2 repo na GitHubie.

## Co powstało

### Meta-projekt (`pattern-library`, public repo)
- **Spec** (`docs/specs/2026-04-07-pattern-library-design.md`) — 12 sekcji, opisuje architekturę, 3 warianty wzorców (v1.2), provenance system, pipeline wyszukiwania
- **Plan** (`docs/plans/2026-04-07-pattern-library-implementation.md`) — 1300+ linii, 3 chunki: scaffold, save-pattern skill, end-to-end weryfikacja
- **Pencil MCP notes** (`docs/plans/pencil-mcp-notes.md`) — pre-discovery findings o tool shapes (batch_get, batch_design, export_nodes, open_document)
- **6 commitów** (latest: `a14da04 docs: formalize pattern variants (v1.2)`)

### Biblioteka (`patterns`, private repo)
- 7 wzorców UI (5 V2 code-only + 1 V1 Pencil + 1 V3 multi-file)
- `INDEX.md` (7 entries) + `TAGS.md` (33 tagi) + `README.md` (z regułą obowiązkowego provenance tag)
- Provenance: 6 gregormedia, 1 specialspace, 0 bravenew

### Skill (lokalny, nie w repo)
- `C:/Users/getrk/.claude/skills/save-pattern.md` (~430 linii)
- 7 kroków atomic workflow + Krok 5b (multi-file detection) + 3 warianty branching + rollback
- Generic file extension support (`.blade.php`, `.tsx`, `.php`, `.html`, `.css`, `.vue`)

## Kluczowe decyzje architektoniczne (podjęte w tej sesji)

### 1. JSON sidecar zamiast standalone .pen
**Powód:** Pencil MCP nie wspiera tworzenia `.pen` plików pod dowolną ścieżką. `batch_design(filePath=X)` ignoruje parametr i operuje na aktywnym dokumencie. Brak `save_as` mechanism.
**Rozwiązanie:** Wzorzec ma `design.json` — JSON snapshot drzewa węzłów z `batch_get`, deserializowany do `batch_design I(...)` przy użyciu wzorca w nowym projekcie.

### 2. Provenance tags zamiast per-client skilli
**Powód:** Pierwotny pomysł zakładał osobne skille per klient (`save-pattern-specialspace`, `save-pattern-bravenew`). Użytkownik wskazał że to nadmierna komplikacja — wystarczy tag.
**Rozwiązanie:** Jedna biblioteka, 3 obowiązkowe provenance tagi (`gregormedia`, `specialspace`, `bravenew`). Filtrowanie w bramce 1 wyszukiwania po provenance pozwala dopasować reuse do reguł licencyjnych klienta.

### 3. Trzy warianty wzorców (v1.2)
**Powód:** Realne wzorce z Laragon często nie mają Pencil designu (są code-only) lub wymagają dodatkowych plików JS/CSS (interaktywne).
**Rozwiązanie:** V1 Pencil-sourced (4 pliki), V2 code-only (3 pliki + sentinel), V3 multi-file (4+ plików, flat lub nested).

### 4. Generic file extension w skill'u
**Powód:** Pierwsza wersja skill'a hardcodowała `implementation.tsx` (mobilny app paradigm). Realny stack to WordPress + Sage + Blade.
**Rozwiązanie:** `implementation.<ext>` gdzie ext dziedziczone z source file. Tabela komentarzy per ext (Blade `{{-- --}}`, JSX `// `, HTML `<!-- -->`, etc.).

## Workflow zwalidowane

| Wariant | Walidacja |
|---|---|
| V1 Pencil-sourced | ✅ hero-split-image-dual-cta z PaniGinekolog (`8QHZT` node, 4 pliki, preview z `export_nodes`) |
| V2 Code-only | ✅ 5 wzorców (text-image, testimonials, numbers, process-steps, why-us) |
| V3 Multi-file flat | ✅ hero-fade-slider z LogoTape (`.blade.php` + `.js`, Swiper init) |
| V3 Multi-file nested | ⏳ jeszcze nie zwalidowane (brak realnego candidate ≥3 plików kodu) |
| save-pattern Skill tool invocation | ⏳ wymaga restart Claude Code (skill stworzony ale nie załadowany w tej sesji) |
| Reader-side find-pattern | ⏳ jeszcze nie istnieje, opisany w spec |
| Rollback test | ⏳ nie zwalidowany na realnym scenariuszu (mechanizm udokumentowany w skill) |

## Provenance map projektów Sage z Laragon (do referencji w przyszłych sesjach)

| Projekt | Theme | Provenance | Sprawdzone w style.css |
|---|---|---|---|
| PaniGinekolog | PaniGinekolog | gregormedia | (potwierdzenie usera) |
| GeoKris | GeoKriss | gregormedia | (potwierdzenie usera) |
| ELD | ELD | gregormedia | (potwierdzenie usera) |
| DBK | DBK | gregormedia | (potwierdzenie usera) |
| LogoTape | LogoTape | gregormedia | (potwierdzenie usera) |
| GregorMediaNew | GregorMedia | gregormedia | ✓ Author: GregorMedia |
| AgroImpex | AgroImpex | gregormedia | ✓ Author: GregorMedia |
| ATM | ATM | gregormedia | ✓ Author: GregorMedia |
| WSK-Growth | WSKGrowth | gregormedia | ✓ Author: GregorMedia |
| Quartz | Quartz | gregormedia | ✓ Author: GregorMedia |
| ProModular | ProModular | gregormedia | ✓ Author: GregorMedia |
| Elderm | Elderm | gregormedia | (default, no Author field) |
| CarOdnowa | CarOdnowa | gregormedia | (default) |
| Chistowscy | chistowscy | gregormedia | (default Sage Starter) |
| Szafari | Szafari | gregormedia | (default Sage Starter) |
| CentralBud | CentralBud | gregormedia | (default Sage Starter) |
| Cortez | Cortez | **specialspace** | (potwierdzenie usera) |
| E-balans | E-balans | **specialspace** | (potwierdzenie usera) |
| LiveFood | (nie sprawdzony) | **specialspace** | (potwierdzenie usera) |
| HCBio | (nie sprawdzony) | **bravenew** | (potwierdzenie usera) |
| InfiniteServices | (nie sprawdzony) | **bravenew** | (potwierdzenie usera) |

## Następne kroki (TODO dla przyszłej sesji)

### High priority
1. **Restart Claude Code** żeby załadować skill `save-pattern` jako wywołanie przez Skill tool
2. **Włącz Laragon hosts** dla DBK + ATM (są offline) — żeby zrobić dev-browser preview backfill
3. **GeoKris mining** — Twój flagowy projekt, jeszcze 0 wzorców stamtąd. 9 home sections do wyboru: hero, co-nas-wyroznia, dla-kogo-pracujemy, o-nas-video, obszar-uslug, realizacje, reviews, kontakt + swiper-slide-realizacje (potencjalny V3)

### Medium priority
4. **Cortez mining** — drugi specialspace, najbogatszy (12 sekcji): about_us×2, blog, cta×2, franczyza×3, marki, partners, strefa_klienta
5. **Quartz hero** — real Swiper hero (potwierdzony slider w grep), kolejny V3 test
6. **Reader-side `find-pattern` skill** — automatyzacja 3-bramkowego pipeline'u wyszukiwania (obecnie regułami w `patterns/README.md`, wykonywane ręcznie przez Claude'a)

### Low priority / backlog
7. **Kompozycje** — `compositions/` katalog jest pusty. Tworzymy gdy 3+ sekcji występuje razem w 2+ projektach.
8. **Cropowanie preview.png** — obecne previews są full-page screenshots. Future improvement: element.screenshot() na konkretne sekcje.
9. **Wersjonowanie wzorców** — gdy wzorzec ewoluuje, trzymamy historię (git tags? semver per pattern?)
10. **Add tagi:** accordion, dropdown, mobile-menu, gallery, pricing-table, contact-form, team-grid (jeśli nowe wzorce tego wymagają)

## Recipe na backfill DBK + ATM po włączeniu hostnames

```bash
cd "C:/Users/getrk/.claude/plugins/cache/dev-browser-marketplace/dev-browser/66682fb0513a/skills/dev-browser"
./server.sh --headless &
sleep 4
npx tsx <<'EOF'
import { connect } from "@/client.js";
const client = await connect();
const targets = [
  { name: "dbk", url: "http://dbk.test/", out: "C:/Users/getrk/patterns/sections/content/numbers-grid-stats/preview.png" },
  { name: "atm", url: "http://atm.test/", out: "C:/Users/getrk/patterns/sections/content/why-us-staggered-cards/preview.png" },
];
for (const t of targets) {
  const page = await client.page(t.name, { viewport: { width: 1440, height: 900 } });
  await page.goto(t.url, { waitUntil: "networkidle", timeout: 15000 });
  await page.waitForTimeout(1500);
  await page.screenshot({ path: t.out, fullPage: true });
}
await client.disconnect();
EOF
```

## Pliki na które warto rzucić okiem w przyszłej sesji
- `pattern-library/docs/specs/2026-04-07-pattern-library-design.md` — sekcja 4.3 (anatomia wzorca + 3 warianty), 4.4 (pipeline wyszukiwania), 12 (backlog)
- `pattern-library/docs/plans/pencil-mcp-notes.md` — kształty wywołań Pencil MCP, kluczowe ograniczenia
- `~/.claude/skills/save-pattern.md` — sekcja "Warianty wzorców" + Krok 5b (multi-file detection)
- `patterns/README.md` — reguły dla Claude'a (3-bramkowy pipeline + provenance rule)
- `patterns/TAGS.md` — kontrolowana lista tagów

## Resume tej konwersacji (jeśli potrzebne)
- Session ID dla `claude -r <id>` — sprawdź w nazwie folderu sesji w `~/.claude/sessions/`
- Lub użyj `/compact` żeby skondensować obecny kontekst i kontynuować lżej
