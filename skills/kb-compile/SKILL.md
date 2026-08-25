---
name: kb-compile
description: Use when asked to compile/update the wiki - "skompiluj kb", "update wiki", "zaciagnij raw do wiki" - or after dropping new material into the vault's raw/threads. Reads raw/ and threads/, extracts concepts, creates or updates wiki/ pages with backlinks. Karpathy-style LLM-as-librarian.
---

# kb-compile - kompilacja wiki z raw + threads

Czyta surowe materialy w kb vault, ekstraktuje koncepty, tworzy/aktualizuje pages w `wiki/`.

## Vault path

```
$KB_PATH/   # domyslnie ~/kb
```

**Git (opcjonalnie)**: jesli vault ma remote, przed compile zawsze `git -C "$KB_PATH" pull --rebase --autostash`. Po compile commit zmian w `wiki/` + push (patrz sekcja "Po kompilacji").

## Tryby

**Full compile** (`kb-compile` bez argumentow):
- Skanuje cale `raw/` i `threads/`
- Aktualizuje `wiki/index.md` + concept files

**Scoped compile** (`kb-compile clients/acme` lub `kb-compile research/solana`):
- Skanuje tylko podfolder
- Aktualizuje subvault wiki

**Single source** (`kb-compile raw/article.md`):
- Inkrementalnie wlacza jeden plik do wiki
- Najtanszy i najczestszy mode

## Algorytm

1. **List sources**:
   - `ls` na `raw/` i `threads/` (lub scoped path)
   - Porownaj z istniejacymi `wiki/*.md` po frontmatter `sources` polu
   - Identyfikuj nowe / zmienione pliki

2. **Read sources**:
   - Czytaj nowe pliki w pelni
   - Czytaj odpowiednie istniejace wiki pages do kontekstu

3. **Extract concepts**:
   - Co to za temat? Czego dotyczy? Jakie kluczowe pojecia?
   - Czy istnieje juz `wiki/<concept>.md`? Jak tak - update, nie nadpisuj.
   - Jak nie - utworz nowy zgodnie z templatka (patrz nizej)

4. **Update wiki page**:
   - Templatka:
     ```markdown
     ---
     title: Concept Name
     created: YYYY-MM-DD
     updated: YYYY-MM-DD
     type: concept
     tags: [...]
     sources:
       - raw/file1.md
       - threads/2026-04-13-foo.md
     ---

     # Concept Name

     ## Definicja
     1-3 zdania.

     ## Kluczowe insighty
     - punkt 1 (skad: [[threads/2026-04-13-foo]])
     - punkt 2

     ## Powiazane
     - [[other-concept]]
     - [[clients/acme]] jak dotyczy

     ## Zrodla
     - raw/file1.md
     - https://external.url
     ```

5. **Update index**:
   - Dodaj nowy concept do `wiki/index.md` pod odpowiednia kategoria
   - Update `## Ostatnio dodane` (max 10 ostatnich)

6. **Backlinks**:
   - Dla kazdego nowego konceptu - sprawdz czy inne wiki pages powinny linkowac do niego
   - Dodaj wzajemne `[[backlinks]]` gdzie sensownie

## Zasady

- **Plaska struktura** - nie tworz deep nestingu w `wiki/`. Maks 1 poziom subfolderow (np. `wiki/clients/<klient>.md`).
- **Merge nad split** - lepiej jedna gruba notatka z sekcjami niz 10 cienkich plikow
- **Nie kasuj danych** - jak source mowi co innego niz wiki, dodaj `## Conflicts` zamiast nadpisac
- **Nie wymyslaj** - jak nie ma w sources, nie pisz w wiki. Brak halucynacji.
- **Path portability (cross-machine)** - wiki bywa czytane na innych maszynach. **NIE** wpisuj lokalnych sciezek (`/Users/...`, `/home/...`, `C:\...`) do wiki. Zamiast tego:
  - URL do repo dla kodu w shared repo
  - Relatywna sciezka do vaulta (`raw/...`, `threads/...`)
  - Znacznik `(lokalnie u <osoba>: apps/foo - NIE w shared repo)` jesli plik jest tylko na jednej maszynie
  - Jesli source (thread) ma lokalna sciezke - zamien na portable podczas compile

## Output

Po skompilowaniu raportuj krotko:
- Ile plikow przetworzono
- Ile nowych konceptow stworzono
- Ile zaktualizowano
- Lista nazw nowych konceptow

Przyklad:
```
kb-compile gotowe.
- 3 nowe sources (2 raw, 1 thread)
- 1 nowy concept: prediction-market-arbitrage
- 2 update: solana-anchor, off-ramp-poland
```

## Lekko ostrozny tryb

Jak zmiana >10 plikow albo nowy concept tam gdzie wiki ma juz duzo - zapytaj usera przed zapisem. Domyslnie: pisz bez pytania.

## Po kompilacji (git sync)

Jesli vault ma remote, po zmianach w `wiki/` commituj i pushuj:

```bash
git -C "$KB_PATH" add wiki/
git -C "$KB_PATH" commit -m "compile: <krotki opis co sie zmienilo>"
git -C "$KB_PATH" push
```

Jesli push fail z powodu non-fast-forward (ktos inny pushowal w miedzyczasie):
`git -C "$KB_PATH" pull --rebase --autostash && git -C "$KB_PATH" push`.
