---
name: kb-dump
description: Use ON-DEMAND when the current conversation has produced insights, decisions, or research worth preserving across sessions. Saves the thread to the kb vault as a markdown file in threads/. A tool for humans using agents - it never fires autonomously in the background. The agent SUGGESTS a save when it sees value (architecture decisions, research deep-dives, project context); the human decides. Triggers - "zapisz watek", "dump to kb", "save thread".
---

# kb-dump - zapis watku do kb vault

Zapisuje aktualny watek rozmowy do knowledge base vault (kb) trzymanego jako katalog z markdownem (zwykle repo gita).

## Vault paths

```
$KB_PATH/threads/   # synteza watku
$KB_PATH/raw/       # raw source material (transkrypty, logi, zrzuty)
```

`KB_PATH` to zmienna srodowiskowa. Gdy nie ustawiona - domyslnie `~/kb`. Nigdy nie hardkoduj sciezki do vaulta w tresci notatek.

**Git (opcjonalnie, ale zalecane)**: jesli vault ma remote (`git -C "$KB_PATH" remote -v` cos zwraca), kazda zmiana musi byc commitowana i pushowana - inaczej reszta zespolu / inne maszyny tego nie zobacza. Bez remote pomijasz kroki gitowe.

Oba foldery sa czytane przez `kb-compile` - synteza + raw trafiaja do wiki.

## Kiedy zapisywac (on-demand, nie autonomicznie)

To narzedzie dla czlowieka uzywajacego agenta. **Nic nie leci samo w tle.** Agent PROPONUJE zapis (jednym zdaniem), user decyduje - albo user prosi wprost.

**Proponuj zapis / zapisuj gdy user poprosi, gdy:**
- Rozmowa zawiera nietrywialne decyzje techniczne lub biznesowe
- User podaje nowy kontekst projektu / klienta
- Wykonano research nad zewnetrznym tematem (artykul, repo, dokumentacja)
- Powstaly nowe pliki / skille / konfigi w wyniku rozmowy
- User jawnie prosi ("zapisz", "dump", "do pamieci")

**NIE proponuj gdy:**
- Trywialne pytania ("jak dziala X?")
- Czysto formatowanie / refactor
- Watek to byl tylko 2-3 wymiany bez substancji

## Format pliku

Nazwa: `YYYY-MM-DD-<slug>.md`

- `<slug>` = 3-6 slow kebab-case, bez znakow diakrytycznych
- Przyklad: `2026-04-13-kb-vault-setup.md`, `2026-04-13-acme-migracja-strategia.md`

Frontmatter:

```yaml
---
title: Krotki tytul
date: YYYY-MM-DD
type: thread
tags: [tag1, tag2]
participants: [user, agent]
related: [clients/<klient>, research/<domena>]  # opcjonalne, jak dotyczy
files_created:  # opcjonalne, lista nowych plikow z tej rozmowy
  - path/to/file
raw_sources:  # opcjonalne, lista raw plikow w raw/ powiazanych z watkiem
  - raw/2026-04-15-slug-mix.txt
---
```

Body sekcje (wszystkie opcjonalne, uzyj co pasuje):

```markdown
# Tytul

## Kontekst
Skad to wzielo. O co user pytal. Jaki problem.

## Co ustalono
Kluczowe decyzje, wnioski, kierunki. Nie streszczaj kazdej wymiany - tylko substancje.

## Co powstalo
Lista plikow / skilli / configow / komend ktore zostaly stworzone albo zmienione.

## Otwarte watki
Co zostalo nierozstrzygniete, do czego wrocic.

## Linki
- Wiki: [[concept-name]]
- Projekt: [[clients/klient]]
- Zewnetrzne URL ktore byly omawiane
```

## Save flow

1. Zdecyduj slug z tematu watku
2. Sprawdz czy plik z dzisiejsza data + slug juz nie istnieje (jak tak - dodaj `-2`, `-3`)
3. **Przed zapisem pociagnij najnowsze zmiany z remote** (jesli vault ma remote):
   `git -C "$KB_PATH" pull --rebase --autostash`
4. **Jesli sa raw source files** (transkrypty, logi, zrzuty) - skopiuj je do `$KB_PATH/raw/` z nazwa `YYYY-MM-DD-<slug>-<rola>.<ext>` (np. `-mix.txt`, `-mic.txt`, `-system.txt`, `-log.txt`)
5. Napisz plik threadu. Jesli dodales raw - wpisz je do `raw_sources:` w frontmatterze i w sekcji `## Linki`
6. **Commit + push** (jesli vault ma remote):
   ```bash
   git -C "$KB_PATH" add threads/ raw/
   git -C "$KB_PATH" commit -m "thread: <slug>"
   git -C "$KB_PATH" push
   ```
7. Konfirm krotko: "watek zapisany: `2026-04-13-slug.md`" + ile raw plikow doszlo do `raw/`

## Kiedy zapisywac raw

**Dropuj raw do `raw/` gdy:**
- Transkrypt spotkania - **glos rozmowcy, jego jezyk, ton i slownictwo sa cennym sygnalem** do pozniejszej analizy (discovery, brief, copy, ocena nastroju)
- Research: artykul, doc page, changelog, transkrypt YouTube - surowy material do pozniejszej ekstrakcji
- Zrzut outputu CLI / logu / bledu, ktory moze byc uzyty do debugowania lub jako dowod
- Kazdy material ktorego synteza w threadzie gubi nuanse, a moze byc pozniej rozparsowana przez `kb-compile` albo `kb-ask`

**NIE dropuj raw gdy:**
- Material jest juz w internecie pod stabilnym URLem - wystarczy link w threadzie
- Material zawiera sekrety / PII / hasla
- Synteza w threadzie zupelnie wystarcza (np. prosty decision log)
- **Ciezszy plik / setki MB** (eksport, dump, binaria) - NIE do `raw/`, bo blotuje git. Wrzuc na zewnetrzny storage, a w threadzie zostaw **link + opis** (schema/liczby). PII trzymaj poza vaultem.

## Po zapisie - compile to WBUDOWANY ostatni krok (nie osobny skill do odpalania)

`kb-dump` i `kb-compile` sa jednym flow z perspektywy usera: powiedzial "kb dump" = chce dump + compile jednym ruchem. **Compile to obowiazkowy ostatni krok tego skilla - odpal go automatycznie, NIE pytaj.**

Po zapisaniu (i pushnieciu) threadu od razu wywolaj skill `kb-compile` w trybie **single-source na swiezo zapisanym threadzie** (`kb-compile threads/<data>-<slug>.md`) - najtanszy tryb, wchlania tylko ten watek do wiki. To NIE jest "autonomiczne dzialanie w tle" (o ktore chodzi w framingu on-demand) - to domkniecie akcji ktora user wlasnie zlecil.

`kb-compile` zostaje osobnym skillem tylko do wywolan solo (np. po wrzuceniu materialu do `raw/` bez dumpa) - logika compile zyje w jednym miejscu i jest reuzywana, nie duplikowana do kb-dump.

## Path portability (KRYTYCZNE - cross-machine)

Kb bywa shared repo, watki czytane na innych maszynach i przez innych ludzi. Sciezki typu `/Users/ktos/projekty/...` w tresci threadu sa **bezuzyteczne dla innych** (nie maja tego katalogu).

### NIE uzywaj w tresci threadu (ani we frontmatter `files_created:`, ani w body):

- Absolutnych sciezek lokalnych (`/Users/...`, `/home/...`, `C:\...`) spoza vaulta

### Uzywaj:

- **URL do repo** dla kodu/plikow w shared repo:
  `https://github.com/<org>/<repo>/tree/main/apps/<app>`
- **Relatywne do vaulta** dla plikow w kb:
  `raw/2026-04-20-slug-mix.txt`, `threads/2026-04-20-foo.md`
- **Znacznik "lokalne u X"** jesli plik istnieje tylko na jednej maszynie:
  `(lokalnie u <osoba>: apps/foo/ - NIE w shared repo)`

### Checklist przed commitem threadu:

- [ ] grep `/Users/|/home/|C:\\` - jesli match poza `$KB_PATH`, zamien
- [ ] Sekcja `## Co powstalo` - lista plikow/projektow -> URL repo albo "lokalnie u X"
- [ ] `files_created:` w frontmatter - te same zasady
- [ ] Czy ktos po `git pull` vaulta dostaje sie do wszystkich zasobow wymienionych w threadzie? (jesli nie - dodaj info "lokalne" albo wrzuc plik do `raw/`)

## Ostroznosc

- Nigdy nie nadpisuj istniejacych watkow ani raw plikow - dodaj suffix
- Nie zapisuj sekretow / API keys / hasel z watku ani z raw plikow - filtruj
- Thread = **synteza**, raw = **surowy material**. Nie wklejaj raw verbatim do threadu - thread ma byc zwiezly, raw lezy obok w `raw/` i jest linkowany z frontmattera
