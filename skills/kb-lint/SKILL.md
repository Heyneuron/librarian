---
name: kb-lint
description: Use for a kb health check or cleanup - "lint kb", "sprzatnij wiki", weekly maintenance. Scans the kb vault for inconsistencies, duplicates, missing concept pages, orphans and unsourced claims. Suggests fixes and applies the safe ones.
---

# kb-lint - health check kb vault

Tygodniowy maintenance kb vault. Karpathy-style "knowledge linting".

## Vault path

```
$KB_PATH/   # domyslnie ~/kb
```

**Git (opcjonalnie)**: przed lintem `git -C "$KB_PATH" pull --rebase --autostash`. Safe fixy ktore zaaplikujesz commituj + pushuj z message `kb-lint: <opis>`.

## Co sprawdza

1. **Sprzecznosci**: ten sam koncept opisany inaczej w roznych pages. Numeryczne niezgodnosci.
2. **Duplikaty**: pages o tym samym, ktore powinny byc zmergowane.
3. **Sieroty**: pages bez backlinks (nikt nie linkuje).
4. **Missing concepts**: encje wymieniane czesto w `raw/`/`threads/` ale bez wlasnej `wiki/<concept>.md`.
5. **Unsourced claims**: stwierdzenia w wiki bez `## Zrodla`.
6. **Stale frontmatter**: `updated` data starsza niz najnowsze source ktore page cytuje.
7. **Index drift**: `wiki/index.md` nie odzwierciedla aktualnych pages.

## Output

Plik `reports/YYYY-MM-DD-kb-lint.md` z sekcjami:

```markdown
# kb-lint - YYYY-MM-DD

## Stats
- N pages w wiki
- N pages w raw
- N threads
- N reports

## Sprzecznosci (N)
- [[page-a]] vs [[page-b]] - opis sprzecznosci

## Duplikaty (N)
- [[page-x]] + [[page-y]] - sugerowany merge

## Sieroty (N)
- [[page-z]] - 0 backlinks. Czy still relevant?

## Missing concepts (N)
- "Anchor PDA" - 5 wzmianek w threads, brak wiki page. Kandydat: `wiki/anchor-pda.md`

## Unsourced claims (N)
- [[page-w]] - "X kosztuje $50" bez zrodla

## Stale (N)
- [[page-v]] - updated 2026-03-01, ale source z 2026-04-10

## Index drift
- 3 pages nie sa w wiki/index.md
- 1 entry w index wskazuje na nieistniejacy plik

## Auto-applied fixes
- Updated wiki/index.md
- Fixed 3 stale `updated` frontmatters
```

## Auto-fix vs propose

**Auto-fix (rob sam):**
- Update `wiki/index.md` zeby odzwierciedlal aktualne pages
- Fix stale `updated` daty
- Dodaj brakujace backlinki gdy oczywiste

**Propose tylko (czekaj na usera):**
- Mergowanie duplikatow
- Tworzenie nowych pages dla missing concepts
- Kasowanie sierot
- Rozwiazywanie sprzecznosci

## Zasady

- **Nigdy nie kasuj danych** bez explicit zgody
- **Nigdy nie nadpisuj sources** w `raw/`/`threads/`
- **Konfliktow nie zgaduj** - oznacz, niech user zdecyduje
