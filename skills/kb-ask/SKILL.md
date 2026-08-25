---
name: kb-ask
description: Use when a question should be answered from the kb vault corpus instead of generic model knowledge. Triggers - "z mojego kb...", "uzyj mojego wiki...", "co wiemy o X w kb", "kb-ask <pytanie>". Reads relevant wiki pages, generates a markdown report saved to reports/ with source links.
---

# kb-ask - Q&A nad kb vault

Pytania nad knowledge base. Czyta wiki, pisze raport do `reports/`.

## Vault path

```
$KB_PATH/   # domyslnie ~/kb
```

**Git (opcjonalnie)**: jesli vault ma remote, przed odpytywaniem `git -C "$KB_PATH" pull --rebase --autostash` zeby czytac najnowsza wersje (ktos inny mogl dodac wpisy). Raport w `reports/` commituj + pushuj po zapisie.

## Algorytm

1. **Locate relevant pages**:
   - Czytaj `wiki/index.md` - pelna mapa konceptow
   - Identyfikuj pages relevantne do pytania
   - Jak pytanie scoped (klient / research domena) - czytaj `clients/<x>/wiki/` lub `research/<x>/wiki/`
   - Tez czytaj linkowane sources w `raw/` i `threads/` jak warto

2. **Read end-to-end**:
   - Pelne pages, nie skrawki
   - Zbieraj cytaty, decyzje, dane numeryczne
   - Sledz backlinks gdy temat sie rozszerza

3. **Synteza**:
   - Odpowiedz oparta TYLKO na wiki + sources. Brak generycznej wiedzy modelu, o ile user nie poprosi.
   - Cytuj zrodla (link do `wiki/<page>` albo `raw/<file>`)
   - Zaznacz luki: "wiki nie pokrywa X, polecam dropnac material o tym"

4. **Save report**:
   - Path: `reports/YYYY-MM-DD-<slug>.md`
   - Frontmatter:
     ```yaml
     ---
     title: Pytanie albo skrocony tytul
     date: YYYY-MM-DD
     type: report
     question: "pelne pytanie usera"
     scope: full|clients/<x>|research/<x>
     sources_read:
       - wiki/concept.md
       - raw/file.md
     tags: [...]
     ---
     ```
   - Body:
     ```markdown
     # Tytul

     ## Pytanie
     > Pelne pytanie usera

     ## Odpowiedz
     Synteza, sekcje, listy. Cytaty z linkami do source.

     ## Zrodla uzyte
     - [[wiki/concept]] - co stamtad wzielo
     - [[raw/file]] - co stamtad wzielo

     ## Luki w wiki
     - czego brakuje
     - co warto dropnac do raw/
     ```

5. **Output do usera**:
   - Krotko w chacie: "raport: `reports/YYYY-MM-DD-slug.md`"
   - Wyswietl 3-5 zdaniowy TLDR z raportu
   - Zaproponuj follow-up pytania

## Zasady

- **Nigdy nie generuj tresci spoza vaulta** o ile user nie powie "uzupelnij z Internetu"
- **Brak chat-tylko odpowiedzi** - zawsze plik. To compounduje.
- **Linkuj wszystko** - `[[wiki/x]]`, `[[raw/y]]` - backlinki (Obsidian i spolka) robia robote
- **Nazwy plikow bez znakow diakrytycznych** - tresc w jezyku vaulta
- **Po raporcie** - sugeruj czy warto sfileowac go z powrotem do `wiki/` (gdy wprowadza nowy insight ktorego tam nie bylo)
